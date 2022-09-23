# DMA 引擎 API 指南[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/client.html#dma-engine-api-guide)

Vinod Koul <vinod dot koul at intel.com>

> 笔记
>
> 对于 async_tx 中 DMA 引擎的使用，请参阅： `Documentation/crypto/async-tx-api.rst`

下面是设备驱动程序编写者如何使用 DMA 引擎的 Slave-DMA API 的指南。这仅适用于从 DMA 使用。

## DMA 使用情况[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/client.html#dma-usage)

从机 DMA 的使用包括以下步骤：

- 分配一个 DMA 从通道
- 设置从机和控制器特定参数
- 获取事务描述符
- 提交交易
- 发出待处理的请求并等待回调通知

这些操作的细节是：

1. 分配一个 DMA 从通道

   通道分配在从属 DMA 上下文中略有不同，客户端驱动程序通常只需要来自特定 DMA 控制器的通道，甚至在某些情况下需要特定通道。使用 dma_request_chan() API 请求通道。

   界面：

   ```
   struct dma_chan *dma_request_chan(struct device *dev, const char *name);
   ```

   它将找到并返回`name`与“开发”设备关联的 DMA 通道。关联是通过 DT、ACPI 或基于板文件的 dma_slave_map 匹配表完成的。

   通过此接口分配的通道对调用者来说是专有的，直到 dma_release_channel() 被调用。

2. 设置从机和控制器特定参数

   下一步总是将一些特定信息传递给 DMA 驱动程序。从 DMA 可以使用的大多数通用信息都在 struct dma_slave_config 中。这允许客户端为外设指定 DMA 方向、DMA 地址、总线宽度、DMA 突发长度等。

   如果某些 DMA 控制器有更多参数要发送，那么它们应该尝试将 struct dma_slave_config 嵌入到它们的控制器特定结构中。如果需要，这使客户端可以灵活地传递更多参数。

   界面：

   ```
   int dmaengine_slave_config(struct dma_chan *chan,
                     struct dma_slave_config *config)
   ```

   请参阅 dmaengine.h 中的 dma_slave_config 结构体定义，了解结构体成员的详细说明。请注意，“方向”成员将离开，因为它与准备调用中给出的方向重复。

3. 获取事务描述符

对于从机使用，DMA 引擎支持的各种从机传输模式是：

- slave_sg：DMA 从/到外设的分散收集缓冲区列表
- dma_cyclic：从外设执行循环 DMA 操作，直到操作明确停止。
- interleaved_dma：这对 Slave 和 M2M 客户端很常见。对于驱动程序可能已经知道设备 fifo 的从地址。可以通过为“dma_interleaved_template”成员设置适当的值来表达各种类型的操作。如果通道支持通过设置 DMA_PREP_REPEAT 传输标志，也可以进行循环交错 DMA 传输。

此传输 API 的非 NULL 返回表示给定交易的“描述符”。

界面：

```
struct dma_async_tx_descriptor *dmaengine_prep_slave_sg(
           struct dma_chan *chan, struct scatterlist *sgl,
           unsigned int sg_len, enum dma_data_direction direction,
           unsigned long flags);

struct dma_async_tx_descriptor *dmaengine_prep_dma_cyclic(
           struct dma_chan *chan, dma_addr_t buf_addr, size_t buf_len,
           size_t period_len, enum dma_data_direction direction);

struct dma_async_tx_descriptor *dmaengine_prep_interleaved_dma(
           struct dma_chan *chan, struct dma_interleaved_template *xt,
           unsigned long flags);
```

外围驱动程序应该在调用 dmaengine_prep_slave_sg() 之前为 DMA 操作映射了 scatterlist，并且必须保持 scatterlist 映射直到 DMA 操作完成。必须使用 DMA 映射分散列表。如果稍后需要同步映射，也必须使用 DMA 调用 dma_sync_*_for_*()。所以，正常的设置应该是这样的：[`struct device`](https://www.kernel.org/doc/html/latest/driver-api/infrastructure.html#c.device)[`struct device`](https://www.kernel.org/doc/html/latest/driver-api/infrastructure.html#c.device)

```
struct device *dma_dev = dmaengine_get_dma_device(chan);

nr_sg = dma_map_sg(dma_dev, sgl, sg_len);
   if (nr_sg == 0)
           /* error */

   desc = dmaengine_prep_slave_sg(chan, sgl, nr_sg, direction, flags);
```

一旦获得描述符，就可以添加回调信息，然后必须提交描述符。一些 DMA 引擎驱动程序可能在成功准备和提交之间持有自旋锁，因此这两个操作紧密配对非常重要。

> 笔记
>
> 尽管 async_tx API 指定完成回调例程不能提交任何新操作，但从机/循环 DMA 并非如此。
>
> 对于从 DMA，在调用回调函数之前，后续事务可能无法提交，因此允许从 DMA 回调准备和提交新事务。
>
> 对于循环 DMA，回调函数可能希望通过 dmaengine_terminate_async() 终止 DMA。
>
> 因此，在调用可能导致死锁的回调函数之前，DMA 引擎驱动程序删除任何锁非常重要。
>
> 请注意，回调将始终从 DMA 引擎小任务中调用，而不是从中断上下文中调用。

**可选：每个描述符元数据**

DMAengine 提供了两种元数据支持方式。

DESC_METADATA_CLIENT

元数据缓冲区由客户端驱动程序分配/提供，并附加到描述符。

```
int dmaengine_desc_attach_metadata(struct dma_async_tx_descriptor *desc,
                              void *data, size_t len);
```

DESC_METADATA_ENGINE

元数据缓冲区由 DMA 驱动程序分配/管理。客户端驱动程序可以询问元数据的指针、最大大小和当前使用的大小，并可以直接更新或读取它。

因为 DMA 驱动程序管理包含元数据的内存区域，所以客户端必须确保在为描述符运行传输完成回调后，它们不会尝试访问或获取指针。如果没有为传输定义完成回调，则在 issue_pending 之后不得访问元数据。换句话说：如果目的是在传输完成后读回元数据，那么客户端必须使用完成回调。

```
void *dmaengine_desc_get_metadata_ptr(struct dma_async_tx_descriptor *desc,
           size_t *payload_len, size_t *max_len);

int dmaengine_desc_set_metadata_len(struct dma_async_tx_descriptor *desc,
           size_t payload_len);
```

客户端驱动程序可以查询是否支持给定模式：

```
bool dmaengine_is_metadata_mode_supported(struct dma_chan *chan,
           enum dma_desc_metadata_mode mode);
```

根据使用的模式，客户端驱动程序必须遵循不同的流程。

DESC_METADATA_CLIENT

- DMA_MEM_TO_DEV / DEV_MEM_TO_MEM：
  1. 准备描述符 (dmaengine_prep_*) 在客户端缓冲区中构造元数据
  2. 使用 dmaengine_desc_attach_metadata() 将缓冲区附加到描述符
  3. 提交转账
- DMA_DEV_TO_MEM：
  1. 准备描述符 (dmaengine_prep_*)
  2. 使用 dmaengine_desc_attach_metadata() 将缓冲区附加到描述符
  3. 提交转账
  4. 传输完成后，附加缓冲区中的元数据应该可用

DESC_METADATA_ENGINE

- DMA_MEM_TO_DEV / DEV_MEM_TO_MEM：
  1. 准备描述符 (dmaengine_prep_*)
  2. 使用 dmaengine_desc_get_metadata_ptr() 获取指向引擎元数据区域的指针
  3. 更新指针处的元数据
  4. 使用 dmaengine_desc_set_metadata_len() 告诉 DMA 引擎客户端已放入元数据缓冲区的数据量
  5. 提交转账
- DMA_DEV_TO_MEM：
  1. 准备描述符 (dmaengine_prep_*)
  2. 提交转账
  3. 传输完成后，使用 dmaengine_desc_get_metadata_ptr() 获取指向引擎元数据区域的指针
  4. 从指针中读出元数据

> 笔记
>
> 当使用 DESC_METADATA_ENGINE 模式时，描述符的元数据区域在传输完成后不再有效（如果使用，则在完成回调返回时有效）。
>
> 不允许混合使用 DESC_METADATA_CLIENT / DESC_METADATA_ENGINE，客户端驱动程序必须使用每个描述符中的任何一种模式。

4. 提交交易

准备好描述符并添加回调信息后，必须将其放置在 DMA 引擎驱动程序挂起队列中。

界面：

```
dma_cookie_t dmaengine_submit(struct dma_async_tx_descriptor *desc)
```

这将返回一个 cookie，可用于通过本文档未涵盖的其他 DMA 引擎调用检查 DMA 引擎活动的进度。

dmaengine_submit() 不会启动 DMA 操作，它只是将其添加到待处理队列中。为此，请参阅步骤 5，dma_async_issue_pending。

> 笔记
>
> 调用后`dmaengine_submit()`提交的传输描述符（）属于 DMA 引擎。因此，客户端必须认为指向该描述符的指针无效。`struct dma_async_tx_descriptor`

5. 发出挂起的 DMA 请求并等待回调通知

挂起队列中的事务可以通过调用 issue_pending API 来激活。如果通道空闲，则启动队列中的第一个事务，随后的事务排队。

在每个 DMA 操作完成后，队列中的下一个启动并触发一个小任务。tasklet 然后将调用客户端驱动程序完成回调例程以进行通知（如果已设置）。

界面：

```
void dma_async_issue_pending(struct dma_chan *chan);
```

### 更多 API[¶](https://www.kernel.org/doc/html/latest/driver-api/dmaengine/client.html#further-apis)

1. 终止 API

```
int dmaengine_terminate_sync(struct dma_chan *chan)
int dmaengine_terminate_async(struct dma_chan *chan)
int dmaengine_terminate_all(struct dma_chan *chan) /* DEPRECATED */
```

这会导致 DMA 通道的所有活动停止，并且可能会丢弃 DMA FIFO 中尚未完全传输的数据。任何未完成的传输都不会调用回调函数。

此功能有两种变体可用。

dmaengine_terminate_async() 可能不会等到 DMA 完全停止或任何正在运行的完整回调完成。但是可以从原子上下文或完整回调中调用 dmaengine_terminate_async()。必须先调用 dmaengine_synchronize()，然后才能安全地释放 DMA 传输访问的内存或释放从完整回调中访问的资源。

dmaengine_terminate_sync() 将在返回之前等待传输和任何正在运行的完整回调完成。但是不能从原子上下文或完整回调中调用该函数。

dmaengine_terminate_all() 已弃用，不应在新代码中使用。

2. Pause API

```
int dmaengine_pause(struct dma_chan *chan)
```

这会暂停 DMA 通道上的活动，而不会丢失数据。

3. Resume API

```
int dmaengine_resume(struct dma_chan *chan)
```

恢复先前暂停的 DMA 通道。恢复当前未暂停的频道无效。

4. 检查 Txn 完成

```
enum dma_status dma_async_is_tx_complete(struct dma_chan *chan,
          dma_cookie_t cookie, dma_cookie_t *last, dma_cookie_t *used)
```

这可用于检查通道的状态。有关此 API 的更完整描述，请参阅 include/linux/dmaengine.h 中的文档。

这可以与 dma_async_is_complete() 和从 dmaengine_submit() 返回的 cookie 结合使用，以检查特定 DMA 事务的完成情况。

> 笔记
>
> 并非所有 DMA 引擎驱动程序都可以为正在运行的 DMA 通道返回可靠信息。建议 DMA 引擎用户在使用此 API 之前暂停或停止（通过 dmaengine_terminate_all()）通道。

5. 同步终止 API

```
void dmaengine_synchronize(struct dma_chan *chan)
```

将 DMA 通道的终止与当前上下文同步。

此函数应在 dmaengine_terminate_async() 之后使用，以将 DMA 通道的终止与当前上下文同步。该函数将在返回之前等待传输和任何正在运行的完整回调完成。

如果 dmaengine_terminate_async() 用于停止 DMA 通道，则必须先调用此函数，然后才能安全地释放先前提交的描述符访问的内存或释放在先前提交的描述符的完整回调中访问的任何资源。

如果 dmaengine_terminate_async() 和此函数之间调用了 dma_async_issue_pending()，则此函数的行为未定义。