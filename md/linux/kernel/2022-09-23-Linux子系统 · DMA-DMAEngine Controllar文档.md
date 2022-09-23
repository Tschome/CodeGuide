# DMAengine 控制器文档[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#dmaengine-controller-documentation)

这本书有助于 DMAengine 内部 API 和 DMAEngine 设备驱动程序编写者指南。

## 硬件介绍[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#hardware-introduction)

大多数从 DMA 控制器具有相同的一般操作原理。

> DMA是Direct Memory Access的缩写，顾名思义，就是绕开CPU直接访问memory的意思。在计算机中，相比CPU，memory和外设的速度是非常慢的，因而在`memory2memory`（或者`memory2dev`）之间搬运数据，非常浪费CPU的时间，造成CPU无法及时处理一些实时事件。因此，工程师们就设计出来一种专门用来搬运数据的器件----DMA控制器，协助CPU进行数据搬运，

它们有给定数量的通道用于 DMA 传输，以及给定数量的请求行。

请求和通道几乎是正交的。通道可用于为任何请求提供多个服务。为简化起见，通道是将进行复制的实体，并请求涉及哪些端点。

请求线路实际上对应于从符合 DMA 条件的设备到控制器本身的物理线路。每当设备想要开始传输时，它都会通过断言该请求线来断言 DMA 请求 (DRQ)。

一个非常简单的 DMA 控制器只考虑一个参数：**传输大小**(`--transfer size`)。在每个时钟周期，它将一个字节的数据从一个缓冲区传输到另一个缓冲区，直到达到传输大小。

这在现实世界中效果不佳，因为从设备可能需要在单个周期内传输特定数量的位。例如，在执行简单的内存复制操作时，我们可能希望传输物理总线允许的尽可能多的数据以最大限度地提高性能，但我们的音频设备可能具有更窄的 FIFO，需要一次精确写入 16 位或 24 位数据. 这就是为什么大多数（如果不是全部）DMA 控制器都可以使用称为**传输宽度**的参数来调整它的原因。

此外，一些 DMA 控制器，无论何时将 RAM 用作源或目标，都可以将内存中的读取或写入分组到缓冲区中，因此不会有很多小内存访问，这不是很有效，你会得到几个更大的转移。这是使用一个称为突发大小的参数来完成的，该参数定义了在控制器将传输拆分为较小的子传输的情况下允许进行多少次单次读取/写入。

> 当传输的源或者目的地是memory的时候，为了提高效率，DMA controller不愿意每一次传输都访问memory，而是在内部开一个buffer，将数据缓存在自己buffer中：
>
> > memory是源的时候，一次从memory读出一批数据，保存在自己的buffer中，然后再一点点（以时钟为节拍），传输到目的地；
> > memory是目的地的时候，先将源的数据传输到自己的buffer中，当累计一定量的数据之后，再一次性的写入memory。
>
> 这种场景下，DMA控制器内部可缓存的**数据量的大小**，称作burst size----另一个参数。

我们理论上的 DMA 控制器将只能进行涉及单个连续数据块的传输。但是，我们通常进行的一些传输不是，并且希望将数据从非连续缓冲区复制到连续缓冲区，这称为分散-聚集。

DMAEngine，至少对于 mem2dev 传输，需要对 scatter-gather 的支持。所以我们这里有两种情况：要么我们有一个不支持它的非常简单的 DMA 控制器，我们必须用软件实现它，或者我们有一个更高级的 DMA 控制器，它在硬件分散中实现-收集(scatter-gather)。

后者通常使用一组要传输的块进行编程，每当开始传输时，控制器都会检查该集合，执行我们在那里编程的任何操作。

该集合通常是表或链表。然后，您将表的地址及其元素数量或列表的第一项推送到 DMA 控制器的一个通道，并且每当断言 DRQ 时，它将通过集合知道从哪里获取来自的数据。

无论哪种方式，此集合的格式完全取决于您的硬件。**每个 DMA 控制器都需要不同的结构，但对于每个块，它们都需要至少源地址和目标地址，是否应该增加这些地址以及我们之前看到的三个参数：突发大小，传输宽度和传输大小**。

最后一件事是，**通常从属设备默认不会发出 DRQ**，**只要您愿意使用 DMA，就必须首先在从属设备驱动程序中启用此功能。**

这些只是一般的内存到内存（也称为 mem2mem）或内存到设备（mem2dev）类型的传输。大多数设备通常支持 dmaengine 支持的其他类型的传输或内存操作，将在本文档后面详细介绍。

## Linux 中的 DMA 支持[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#dma-support-in-linux)

`Historically, DMA controller drivers have been implemented using the async TX API, to offload operations such as memory copy, XOR, cryptography, etc., basically any memory to memory operation.`

从**历史上**看，DMA 控制器驱动程序是使用异步 TX API 实现的，以卸载内存复制、XOR、加密等操作，基本上是**任何内存到内存**的操作。

> DMA原来一开始只是支持mem2mem的功能

随着时间的推移，出现了对内存到设备传输的需求，并且扩展了 dmaengine。如今，异步 TX API 被编写为 dmaengine 之上的一层，并充当**客户端**。尽管如此，dmaengine 在某些情况下仍能容纳该 API，并做出了一些设计选择以确保其保持兼容。

有关 Async TX API 的更多信息，请查看[Asynchronous Transfers/Transforms API](https://www.kernel.org/doc/html/latest/crypto/async-tx-api.html)中的相关文档文件。

## DMAEngine API[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#dmaengine-apis)

### `struct dma_device`初始化[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#struct-dma-device-initialization)

就像任何其他内核框架一样，整个 DMAEngine 注册依赖于驱动程序填充结构并针对框架进行注册。在我们的例子中，该结构是 dma_device。

您需要在驱动程序中做的第一件事是分配此结构。任何常用的内存分配器都可以，但您还需要在其中初始化一些字段：

- `channels`: 应该使用 INIT_LIST_HEAD 宏初始化为一个列表，例如
- `src_addr_widths`：应包含支持的源传输宽度的位掩码
- `dst_addr_widths`：应包含支持的目标传输宽度的位掩码
- `directions`：应包含支持的从方向的位掩码（即不包括 mem2mem 传输）
- `residue_granularity`：报告给 dma_set_residue 的传输残差的粒度。这可以是：
  - Descriptor 描述符：您的设备不支持任何类型的残留报告。框架只会知道特定的事务描述符已完成。
  - Segment 分段：您的设备能够报告已传输的块
  - Burst 突发：您的设备能够报告已传输的突发
- `dev`: 应该持有指向与当前驱动程序实例相关联的指针。`struct device`

### 支持的交易类型[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#supported-transaction-types)

接下来您需要设置您的设备（和驱动程序）支持的事务类型。

我们`dma_device structure`有一个名为 `cap_mask` 的字段，其中包含支持的各种类型的事务，您需要使用 dma_cap_set 函数修改此掩码，并根据您作为参数支持的事务类型使用各种标志。

所有这些功能都在, `dma_transaction_type enum``include/linux/dmaengine.h`里

目前，可用的类型有：

- DMA_MEMCPY
  - 该设备能够进行内存到内存的复制
  - 无论源和目标组合块的总大小是多少，都只会传输两者中最小的字节数。这意味着两个列表中分散收集缓冲区的数量和大小不必相同，并且该操作在功能上等效于 a `strncpy`，其中`count`参数等于两个分散收集列表缓冲区的最小总大小。
  - 它通常用于在主机内存和内存映射的 GPU 设备内存之间复制像素数据，例如在现代 PCI 视频显卡上。最直接的例子是 OpenGL API 函数 `glReadPielx()`，它可能需要从本地设备内存到主机内存的巨大帧缓冲区的逐字复制。
- DMA_XOR
  - 该设备能够对内存区域执行 XOR 操作
  - 用于加速 XOR 密集型任务，例如 RAID5
- DMA_XOR_VAL
  - 该设备能够使用 XOR 算法对内存缓冲区执行奇偶校验。
- DMA_PQ
  - 该设备能够执行 RAID6 P+Q 计算，P 是简单的 XOR，Q 是 Reed-Solomon 算法。
- DMA_PQ_VAL
  - 该设备能够使用 RAID6 P+Q 算法对内存缓冲区执行奇偶校验。
- DMA_MEMSET
  - 设备能够使用提供的模式填充内存
  - 该模式被视为单字节有符号值。
- DMA_INTERRUPT
  - 该设备能够触发将产生周期性中断的虚拟传输
  - 由客户端驱动程序用于注册回调，该回调将通过 DMA 控制器中断定期调用
- DMA_PRIVATE
  - 这些设备仅支持从属传输，因此不适用于异步传输。
- DMA_ASYNC_TX
  - 不能由设备设置，需要时由框架设置
  - 托多：这是关于什么的？
- DMA_SLAVE
  - 该设备可以处理设备到内存的传输，包括分散-聚集传输。
  - 虽然在 mem2mem 案例中，我们有两种不同的类型来处理要复制的单个块或它们的集合，但在这里，我们只有一个应该处理两者的事务类型。
  - 如果要传输单个连续的内存缓冲区，只需构建一个只有一项的分散列表。
- DMA_CYCLIC
  - 该设备可以处理循环传输。
  - 循环传输是块集合将在自身上循环的传输，最后一个项目指向第一个。
  - 它通常用于音频传输，您希望在单个环形缓冲区上进行操作，您将使用音频数据填充该缓冲区。
- DMA_INTERLEAVE
  - 该设备支持交错传输。
  - 这些传输可以将数据从非连续缓冲区传输到非连续缓冲区，而 DMA_SLAVE 可以将数据从非连续数据集传输到连续目标缓冲区。
  - 它通常用于 2d 内容传输，在这种情况下，您希望将一部分未压缩数据直接传输到显示器进行打印
- DMA_COMPLETION_NO_ORDER
  - 该设备不支持按顺序完成。
  - 如果设备正在设置此功能，则驱动程序应为 device_tx_status 返回 DMA_OUT_OF_ORDER。
  - 如果设备导出此功能，则所有 cookie 跟踪和检查 API 都应视为无效。
  - 此时，这与 dmatest 的轮询选项不兼容。
  - 如果设置了此上限，建议用户为发送到 DMA 设备的每个描述符提供唯一标识符，以便正确跟踪完成情况。
- DMA_REPEAT
  - 该设备支持重复传输。由 DMA_PREP_REPEAT 传输标志指示的重复传输类似于循环传输，因为它在结束时会自动重复，但可以另外由客户端替换。
  - 此功能仅限于交错传输，因此如果未设置 DMA_INTERLEAVE 标志，则不应设置此标志。此限制基于 DMA 客户端的当前需求，如果需要，将来应添加对其他传输类型的支持。
- DMA_LOAD_EOT
  - 该设备支持通过设置 DMA_PREP_LOAD_EOT 标志对新传输进行排队来替换传输结束 (EOT) 时的重复传输。
  - 如果需要，将来会根据 DMA 客户端的需要添加对在另一个点替换当前正在运行的传输（例如突发结束而不是传输结束）的支持。

这些不同的类型也会影响源地址和目标地址随时间的变化。

指向 RAM 的地址通常在每次传输后递增（或递减）。如果是环形缓冲区，它们可能会循环（DMA_CYCLIC）。指向设备寄存器（例如 FIFO）的地址通常是固定的。

### 每个描述符元数据支持[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#per-descriptor-metadata-support)

一些数据移动架构（DMA 控制器和外围设备）使用与事务相关的元数据。DMA 控制器的作用是同时传输有效负载和元数据。元数据本身不被 DMA 引擎使用，但它包含外设或来自外设的参数、密钥、向量等。

DMAengine 框架提供了一种通用方法来促进描述符的元数据。根据架构，DMA 驱动程序可以实现其中一种或两种方法，由客户端驱动程序选择使用哪一种。

- DESC_METADATA_CLIENT

  元数据缓冲区由客户端驱动程序分配/提供，并附加（通过 dmaengine_desc_attach_metadata() 帮助程序到描述符。

  从 DMA 驱动程序中，该模式预期以下内容：

  - DMA_MEM_TO_DEV / DEV_MEM_TO_MEM

    来自提供的元数据缓冲区的数据应准备好供 DMA 控制器与有效载荷数据一起发送。通过复制到硬件描述符或高度耦合的数据包。

  - DMA_DEV_TO_MEM

    传输完成后，DMA 驱动程序必须先将元数据复制到客户端提供的元数据缓冲区，然后再通知客户端完成。传输完成后，DMA 驱动程序不得接触客户端提供的元数据缓冲区。

- DESC_METADATA_ENGINE

  元数据缓冲区由 DMA 驱动程序分配/管理。客户端驱动程序可以询问元数据的指针、最大大小和当前使用的大小，并可以直接更新或读取它。dmaengine_desc_get_metadata_ptr() 和 dmaengine_desc_set_metadata_len() 作为辅助函数提供。

  从 DMA 驱动程序中，该模式预期以下内容：

  - get_metadata_ptr()

    应该返回元数据缓冲区的指针、元数据缓冲区的最大大小以及缓冲区中当前使用/有效（如果有）的字节。

  - set_metadata_len()

    客户端在将元数据放入缓冲区后调用它，让 DMA 驱动程序知道提供的有效字节数。

  注意：由于客户端将在完成回调中请求元数据指针（在 DMA_DEV_TO_MEM 情况下），因此 DMA 驱动程序必须确保在调用回调之前未释放描述符。

### 设备操作[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#device-operations)

我们的 dma_device 结构还需要一些函数指针来实现实际的逻辑，现在我们已经描述了我们能够执行的操作。

我们必须在那里填写并因此必须实现的功能显然取决于您报告为支持的交易类型。

- `device_alloc_chan_resources`
- `device_free_chan_resources`
  - 每当驱动程序调用`dma_request_channel`或`dma_release_channel`在与该驱动程序关联的通道上第一次/最后一次调用这些函数时，都会调用这些函数 。
  - 他们负责分配/释放所有需要的资源，以使该通道对您的驱动程序有用。
  - 这些函数可以休眠。
- `device_prep_dma_*`
  - 这些功能与您之前注册的功能相匹配。
  - 这些函数都获取与正在准备的传输相关的缓冲区或分散列表，并应从中创建硬件描述符或硬件描述符列表
  - 这些函数可以从中断上下文中调用
  - 您可能进行的任何分配都应该使用 GFP_NOWAIT 标志，以免潜在地休眠，但也不会耗尽紧急池。
  - 驱动程序应尝试在探测时预先分配在传输设置期间可能需要的任何内存，以避免对 nowait 分配器施加太大压力。
  - 它应该返回一个唯一的实例 ，进一步代表这个特定的传输。`dma_async_tx_descriptor structure`
  - 这个结构可以使用函数初始化 `dma_async_tx_descriptor_init`。
  - 您还需要在此结构中设置两个字段：
    - flags: TODO: 是否可以由驱动程序本身修改，还是应该始终是参数中传递的标志
    - tx_submit：指向您必须实现的函数的指针，该函数应该将当前事务描述符推送到待处理队列，等待调用 issue_pending。
  - 在这个结构中，函数指针 callback_result 可以被初始化，以便通知提交者事务已经完成。在前面的代码中，已经使用了函数指针回调。但是，它不为交易提供任何状态，将被弃用。定义为 `dmaengine_result`传递给 callback_result 的结果结构有两个字段：
    - 结果：这提供了定义的传输结果 `dmaengine_tx_result`。成功或某些错误条件。
    - 残留：为那些支持残留的传输提供残留字节。
- `device_issue_pending`
  - 获取待处理队列中的第一个事务描述符，并开始传输。每当该传输完成时，它应该移动到列表中的下一个事务。
  - 该函数可以在中断上下文中调用
- `device_tx_status`
  - 应该报告给定通道上剩余的字节数
  - 应该只关心作为参数传递的事务描述符，而不是给定通道上当前活动的描述符
  - tx_state 参数可能为 NULL
  - 应该使用 dma_set_residue 报告它
  - 在循环传输的情况下，它应该只考虑循环缓冲区的总大小。
  - 如果设备不支持按顺序完成并且正在无序完成操作，则应返回 DMA_OUT_OF_ORDER。
  - 该函数可以在中断上下文中调用。
- device_config
  - 使用作为参数给出的配置重新配置通道
  - 此命令不应同步执行，也不应在任何当前排队的传输上执行，而应仅在后续传输上执行
  - 在这种情况下，该函数将接收一个`dma_slave_config` 结构指针作为参数，它将详细说明要使用的配置。
  - 尽管该结构包含一个方向字段，但该字段已被弃用，取而代之的是提供给 prep_* 函数的方向参数
  - 此调用仅对从属操作是强制性的。不应为 memcpy 操作设置或期望设置此项。如果驱动程序同时支持这两者，它应该只将此调用用于从属操作，而不用于 memcpy 操作。
- device_pause
  - 暂停频道上的传输
  - 该命令应该在通道上同步操作，立即暂停给定通道的工作
- device_resume
  - 恢复通道上的传输
  - 该命令应该在通道上同步操作，立即恢复给定通道的工作
- device_terminate_all
  - 中止通道上所有未决和正在进行的传输
  - 对于中止的传输，不应调用完整的回调
  - 可以从原子上下文或描述符的完整回调中调用。一定不能睡觉。驱动程序必须能够正确处理此问题。
  - 终止可能是异步的。驱动程序不必等到当前活动的传输完全停止。请参阅 device_synchronize。
- device_synchronize
  - 必须将通道的终止与当前上下文同步。
  - 必须确保 DMA 控制器不再访问先前提交的描述符的内存。
  - 必须确保先前提交的描述符的所有完整回调都已完成运行，并且没有计划运行。
  - 可以睡觉了。

## 杂项说明[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#misc-notes)

（应该记录的东西，但真的不知道把它们放在哪里）

```
dma_run_dependencies
```

- 应该在异步 TX 传输结束时调用，并且在从传输情况下可以忽略。
- 确保在将相关操作标记为完成之前运行相关操作。

dma_cookie_t

- 它是一个 DMA 事务 ID，会随着时间的推移而增加。
- `virt-dma` 自从引入它以来，它就不再真正相关了。

DMA_CTRL_ACK

- 如果清除，则在客户端确认收到之前，提供者不能重用描述符，即有机会建立任何依赖链
- 这可以通过调用 async_tx_ack() 来确认
- 如果设置，并不意味着描述符可以重复使用

DMA_CTRL_REUSE

- 如果设置，描述符可以在完成后重用。如果设置了这个标志，它不应该被提供者释放。
- `dmaengine_desc_set_reuse()`应通过调用将设置 DMA_CTRL_REUSE的描述符来准备重用 。
- `dmaengine_desc_set_reuse()`只有当通道支持能力所展示的可重用描述符时才会成功
- 因此，如果设备驱动程序想要在 2 次传输之间跳过 `dma_map_sg()`和`dma_unmap_sg()`，因为没有使用 DMA 的数据，它可以在传输完成后立即重新提交传输。
- 描述符可以通过几种方式释放
  - `dmaengine_desc_clear_reuse()`通过调用和提交最后一个 txn 来清除 DMA_CTRL_REUSE
  - 显式调用`dmaengine_desc_free()`，只有在已经设置了 DMA_CTRL_REUSE 时才能成功
  - 终止频道
- DMA_PREP_CMD
  - 如果设置，客户端驱动程序会告诉 DMA 控制器在 DMA API 中传递的数据是命令数据。
  - 命令数据的解释是 DMA 控制器特定的。它可用于向其他外围设备/寄存器读取/寄存器写入发出命令，其中描述符的格式应与普通数据描述符不同。
- DMA_PREP_REPEAT
  - 如果设置，传输将在结束时自动重复，直到新传输在具有 DMA_PREP_LOAD_EOT 标志的同一通道上排队。如果要在通道上排队的下一个传输没有设置 DMA_PREP_LOAD_EOT 标志，则将重复当前传输，直到客户端终止所有传输。
  - 仅当通道报告 DMA_REPEAT 功能时才支持此标志。
- DMA_PREP_LOAD_EOT
  - 如果设置，则传输将在传输结束时替换当前正在执行的传输。
  - 这是非重复传输的默认行为，因此为非重复传输指定 DMA_PREP_LOAD_EOT 不会有任何区别。
  - 当使用重复传输时，DMA 客户端通常需要在所有传输上设置 DMA_PREP_LOAD_EOT 标志，否则通道将继续重复上一次重复传输并忽略正在排队的新传输。未能设置 DMA_PREP_LOAD_EOT 将显示为通道卡在上一次传输中。
  - 仅当通道报告 DMA_LOAD_EOT 功能时才支持此标志。

## 一般设计说明[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/provider.html#general-design-notes)

您将看到的大多数 DMAEngine 驱动程序都基于类似的设计，在处理程序中处理传输中断的结束，但将大部分工作推迟到一个小任务中，包括在前一次传输结束时开始新的传输。

这是一个相当低效的设计，因为传输间延迟不仅是中断延迟，还有 tasklet 的调度延迟，这会使通道处于空闲状态，从而降低全局传输速率。

你应该避免这种做法，而不是在你的 tasklet 中选择一个新的传输，而是将该部分移动到中断处理程序，以便有一个更短的空闲窗口（无论如何我们都无法真正避免）。



