#高级话题

本文档描述关于极光推送的一些高级的、背后的逻辑。

###消息的生命周期

当一条消息从开发者AppServer（通过API调用）或者Portal上通过 JPush 推送后，不意味着这条消息马上就成功地推送到了用户设备上。实际上，这只是意味着，这条消息被极光推送服务器接收，进入准备推送状态。这条消息被接收后会怎么样，还要依赖于多个因素。

理想情况下，当用户设备与服务器正在保持连接，也没有其他导致这条消息会被丢弃的因素存在，则用户马上会收到这条消息。

接下来，看 time_to_live 字段的指定情况。如果为 0 ，表示该消息不保存离线，即如果用户当前不在线，则用户就会一直收不到该消息。time_to_live 大于 0 时会保留指定时长的离线消息时间。

之后看 override_msg_id 是否被指定。如果新的一条消息，其 override_msg_id 与离线消息里已经存在的一条相同，则这条新的消息会覆盖老的消息。


####离线消息

JPush 当前默认支持 5 条离线消息。如有特殊需求需要更多，请联系商务提出需求。

Android 平台上，用户上线时，会把离线消息推送下去。

iOS 应用内推送，与 Android 情况相同。

iOS 通知，由于是直接基于 Apple 平台本身的推送通道 APNs，由 APNs 的离线消息策略来决定（最后一条）。

离线消息默认的保留时长是 1  天。

API 2.0 提供一个 time_to_live 字段，允许调用推送消息时，设置保留的时长。以秒为单位。如果为 0 表示不保留离线消息，即只对在线的用户推送消息。最长为 10 天。



###消息覆盖

JPush 基于 override_msg_id 来定义消息覆盖逻辑。

即对于同一个应用，如果新的一条消息其 override_msg_id 存在并与老的一条消息相同，则会被认为是要覆盖老的消息。

这个情况对于是否保存离线消息有所不同：

+ 如果客户端已经收到，用户已经点击打开通知：则新的消息（override_msg_id 存在的消息）还是会显示在通知栏，用户看到的是有新的一条消息。
+ 如果客户端已经收到，但还在通知栏：则新的消息会在通知栏上覆盖老的消息。用户看到新的消息内容。
+ 如果客户端还未收到，消息还在离线消息里，则：新的消息会覆盖老的消息，用户不会收到老的消息。
 

###消息删除

一般来说，一条消息推送出来后，如果某用户在线，则他会立即收到，所以没有反悔（删除）的机会了。

JPush 提供消息覆盖的机制，使得你可以有机会可以更新暂时离线的用户最后会收到的消息。

JPush 暂未开放删除 “离线消息” 的机制与接口。如果特殊原因有需求，请发邮件到 [support@jpush.cn](mailto:support@jpush.cn) 由极光推送支持人员来操作。



###推送顺序

极光推送不保证：推送消息的顺序，客户端也以相同的顺序接收到。

但是大多数时候用户收到的消息是按顺序推送的，即使保存离线消息也是有顺序的。
