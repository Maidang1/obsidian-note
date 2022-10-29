# DASH

# **协议介绍**

DASH 全称是 Dynamic Adaptive Streaming over HTTP，即基于HTTP的动态自适应的比特率流。DASH和HLS协议类似，都是将音视频流分割成小块，通过 HTTP 协议进行传输，客户端得到之后进行播放。不同的是 DASH 支持 MPEG-2 TS、MP4等多种媒体格式。

# **协议分析**

DASH 协议的核心都在其 Manifest 文件，DASH的 Manifest 文件名为 Media Presentation Descrption(MPD)，使用 XML 格式，对音视频流作了多个维度的划分。下面是 MPD 文件的基本格式。

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268331042-c05434df-f05b-425f-bbc0-fd576e8aed3b.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268331042-c05434df-f05b-425f-bbc0-fd576e8aed3b.png)

如上图所示，MPD 文件的结构由外向内分别是 Period（周期）->AdaptationSet（自适应子集）->Representation（码流）→Segment（片段）。

-   **Period（周期）：**一条完整的 mpeg dash 码流可能由一个或多个 Period 构成，每个 Period 代表某一个时间段。比如某条码流有 60 秒时间，Period1 从 0-15 秒，Period2 从 16 秒到 40 秒，Period3 从41秒到60秒。同一个 Period 内，意味着可用的媒体内容及其各个可用码率（Representation）不会发生变更。直播情况下，需要周期地去服务器更新 MPD 文件，服务器可能会移除旧的已经过时的Period ,或是添加新的 Period。新的 Period 中可能会添加新的可用码率或去掉上一个 Period 中存在的某些码率, 即上面的 Representation 字段。
-   **Adaptation Set（自适应子集）：**一个 Period 由一个或者多个 Adaptationset 组成，一组可供切换的不同码率的码流组合成一个自适应子集。
-   **Representation（码流）：**每个Adaptation Set 包含了一个或者多个 Representation，每个Representation 代表一路音频流或视频流。同一个 Adaptation Set的多个 Representation 之间代表他们由同一路源流产生的不同的码率、分辨率、帧率等等的码流。
-   **Segment（片段）：**每个 Representation 包含多个 Segment。与 HLS 类似，每个 Segment 代表一小段音频或视频数据，其中 DASH Segment 常用的载体是使用 fmp4 格式。

# **下载播放流程**

1.  下载 MPD 文件，解析 DASH 相关信息；
2.  下载视频的 Initialization Segment 和音频的 Initialization Segment；
3.  下载视频的第一个分片，下载音频的第一个分片；
4.  当视频和音频的第一个分片都下载完，播放器内部再进行一些相关处理后，就可以开始播放出画面。后续就是不断轮询更新 MPD 文件和下载后续的音频和视频分片。

# 举个例子

B站模版里面的信息

```json
{
            id: 112,
            baseUrl:
              '<https://cn-hbcd-cu-02-07.bilivideo.com/upgcxcode/40/19/800231940/800231940_nb3-1-30112.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1660745710&gen=playurlv2&os=bcache&oi=0&trid=000059f9c7d637744ad1b9179ef154c14735u&mid=427444426&platform=pc&upsig=a61a86cbb4cf2f98f62f4fbd1430b230&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&cdnid=7880&bvc=vod&nettype=0&orderid=0,3&agrr=0&bw=627908&logo=80000000>',
            base_url:
              '<https://cn-hbcd-cu-02-07.bilivideo.com/upgcxcode/40/19/800231940/800231940_nb3-1-30112.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1660745710&gen=playurlv2&os=bcache&oi=0&trid=000059f9c7d637744ad1b9179ef154c14735u&mid=427444426&platform=pc&upsig=a61a86cbb4cf2f98f62f4fbd1430b230&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&cdnid=7880&bvc=vod&nettype=0&orderid=0,3&agrr=0&bw=627908&logo=80000000>',
            backupUrl: [
              '<https://upos-sz-mirrorcos.bilivideo.com/upgcxcode/40/19/800231940/800231940_nb3-1-30112.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1660745710&gen=playurlv2&os=cosbv&oi=0&trid=59f9c7d637744ad1b9179ef154c14735u&mid=427444426&platform=pc&upsig=890db498caeb5d12ea4bc18cb6c1ea26&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&bvc=vod&nettype=0&orderid=1,3&agrr=0&bw=627908&logo=40000000>',
              '<https://upos-sz-mirrorcos.bilivideo.com/upgcxcode/40/19/800231940/800231940_nb3-1-30112.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1660745710&gen=playurlv2&os=cosbv&oi=0&trid=59f9c7d637744ad1b9179ef154c14735u&mid=427444426&platform=pc&upsig=890db498caeb5d12ea4bc18cb6c1ea26&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&bvc=vod&nettype=0&orderid=2,3&agrr=0&bw=627908&logo=40000000>',
            ],
            backup_url: [
              '<https://upos-sz-mirrorcos.bilivideo.com/upgcxcode/40/19/800231940/800231940_nb3-1-30112.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1660745710&gen=playurlv2&os=cosbv&oi=0&trid=59f9c7d637744ad1b9179ef154c14735u&mid=427444426&platform=pc&upsig=890db498caeb5d12ea4bc18cb6c1ea26&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&bvc=vod&nettype=0&orderid=1,3&agrr=0&bw=627908&logo=40000000>',
              '<https://upos-sz-mirrorcos.bilivideo.com/upgcxcode/40/19/800231940/800231940_nb3-1-30112.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&nbs=1&deadline=1660745710&gen=playurlv2&os=cosbv&oi=0&trid=59f9c7d637744ad1b9179ef154c14735u&mid=427444426&platform=pc&upsig=890db498caeb5d12ea4bc18cb6c1ea26&uparams=e,uipk,nbs,deadline,gen,os,oi,trid,mid,platform&bvc=vod&nettype=0&orderid=2,3&agrr=0&bw=627908&logo=40000000>',
            ],
            bandwidth: 5020135,
            mimeType: 'video/mp4',
            mime_type: 'video/mp4',
            codecs: 'avc1.640032',
            width: 1920,
            height: 1080,
            frameRate: '25',
            frame_rate: '25',
            sar: '1:1',
            startWithSap: 1,
            start_with_sap: 1,
            SegmentBase: {
              Initialization: '0-979',
              indexRange: '980-3171',
            },
            segment_base: {
              initialization: '0-979',
              index_range: '980-3171',
            },
            codecid: 7,
}
```

可以看到里面有一些关键的信息，包括 segmentbase, indexRange Initialization。 我们在查看使用 MPD 文件描述的视频分片信息，发现这个 JSON 文件也描述了一个视频分片应该具有的全部的信息。

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268362011-8fca3e16-6de2-496d-9f2c-235cb067cebb.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268362011-8fca3e16-6de2-496d-9f2c-235cb067cebb.png)

同时查看 dash.js 的 API，发现我们不仅仅可以传入 MPD 的文件，也可以传入一个 object 进去。由此可以推测，B站将这些信息写入模版中，然后从模版中读取信息，然后传入 dash.js 进行播放。

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268361963-b799323a-5906-423d-9611-9ae91dd68377.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268361963-b799323a-5906-423d-9611-9ae91dd68377.png)

Segmentbase 的形式是用 initialization 来定位视频的初始信息，用 index_range 来定位每个分片对应的 range 信息。在 HLS 中 可以通过 EXT-X-BYTERANGE 做到类似的效果。

# HLS

# **协议介绍**

**HTTP Live Streaming（缩写是HLS）**是一个由苹果公司提出的基于HTTP的流媒体网络传输协议。

是苹果公司QuickTime X和iPhone软件系统的一部分。它的工作原理是把整个流分成一个个小的基于HTTP的文件来下载，每次只下载一些。当媒体流正在播放时，客户端可以选择从许多不同的备用源中以不同的速率下载同样的资源，允许流媒体会话适应不同的数据速率。在开始一个流媒体会话时，客户端会下载一个包含元数据的extended M3U (m3u8)playlist文件，用于寻找可用的媒体流。HLS只请求基本的HTTP报文，与实时传输协议（RTP)不同，HLS可以穿过任何允许HTTP数据通过的防火墙或者代理服务器。它也很容易使用内容分发网络来传输媒体流。

# 协议分析

![https://cdn.nlark.com/yuque/0/2022/jpeg/1598475/1661268362024-d8935a66-8dba-4a97-bde5-6bc4a4660930.jpeg](https://cdn.nlark.com/yuque/0/2022/jpeg/1598475/1661268362024-d8935a66-8dba-4a97-bde5-6bc4a4660930.jpeg)

上面是 HLS 整体架构图， 可以看出，总共有三个部分：Server，CDN，Client。

其实，HLS 协议的主要内容是关于 M3U8 这个文本协议的。

简单的 Media Playlist：

```jsx
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:8
#EXT-X-MEDIA-SEQUENCE:2680
#EXTINF:7.975,
<https://priv.example.com/fileSequence2680.ts>
#EXTINF:7.941,
<https://priv.example.com/fileSequence2681.ts>
#EXTINF:7.975,
<https://priv.example.com/fileSequence2682.ts>
```

包含多种比特率的 Master Playlist：

```jsx
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=1280000
<http://example.com/low.m3u8>
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=2560000
<http://example.com/mid.m3u8>
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=7680000
<http://example.com/hi.m3u8>
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=65000,CODECS="mp4a.40.5"
<http://example.com/audio-only.m3u8>
```

-   HLS 通过 URI(RFC3986) 指向的一个 Playlist 来表示一个媒体流。
-   一个 Playlist 可以是一个 Media Playlist 或者 Master Playlist，使用 UTF-8 编码的文本文件，包含一些 URI 跟描述性的 tags。
-   一个 Media Playlist 包含一个 Media Segments 列表,当顺序播放时，能播放整个完整的流。
-   要想播放这个 Playlist，客户端需要首先下载他，然后播放里面的每一个 Media Segment。

# 播放流程

1.  客户端通过 URI 获取 Playlist. 如果是 Master Playlist, 客户端可以选择一个 Variant Stream 来播放.
2.  客户端检查 EXT-X-VERSION 版本是否满足.
3.  客户端应该忽略不可识别的 tags, 忽略不可识别的属性键值对.
4.  加载 Media Playlist file.
5.  播放 Media Playlist file.
6.  重加载 Media Playlist file.
7.  决定下一次要加载的 Media Segment.

# m4s 文件

M4S 文件是使用 MPEG-DASH 视频流技术流式传输的视频片段。它包含表示视频片段的二进制数据。作为视频第一段的 M4S 文件还包含初始化数据，允许媒体播放器识别并开始播放视频。类似于切割过的 mp4 文件。一个 m4s 文件可以播放。

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268362169-b9c1194c-a15a-4c6d-8021-0ff2731518de.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268362169-b9c1194c-a15a-4c6d-8021-0ff2731518de.png)

# mp4

MP4文件由许多box组成，每个box包含不同的信息， 这些box以树形结构的方式组织。

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268362363-9a10bdf5-63c8-4cc7-92e5-9733294d06a6.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268362363-9a10bdf5-63c8-4cc7-92e5-9733294d06a6.png)

-   box由header和body组成，header指明box的size和type。size是包含box header的整个box的大小。
-   box type，通常是4个ASCII码的字符如“ftyp”、“moov”等，这些box type都是已经预定义好的，表示固定的含义。如果是“uuid”，表示该box为用户自定义扩展类型，如果box type是未定义的，应该将其忽略。
-   如果header中的size为1，则表示box长度需要更多的bits位来描述，在后面会有一个64bits位的largesize用来描述box的长度。如果size为0，表示该box为文件的最后一个box，文件结尾（同样只存在于“mdat”类型的box中）。
-   box中可以包含box，这种box称为container box。

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268363602-e99a40c9-c735-4127-9a48-bfe8d114578e.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268363602-e99a40c9-c735-4127-9a48-bfe8d114578e.png)

-   **ftyp**：文件类型。描述遵从的规范的版本。
-   **moov box**：媒体的metadata信息。
-   **mdat**：具体的媒体数据。

# fmp4 vs mp4

![https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268363857-530fbbda-13cd-47ce-89d0-1a5321f924db.png](https://cdn.nlark.com/yuque/0/2022/png/1598475/1661268363857-530fbbda-13cd-47ce-89d0-1a5321f924db.png)

-   普通mp4的时长、内容通常是固定的。FMp4 时长、内容通常不固定，可以边生成边播放；
-   普通mp4完整的 metadata 都在 moov 里，需要加载完 moov box 后，才能对 mdat 中的媒体数据进行解码渲染；
-   fmp4 中，媒体数据的 metadata 在 moof box 中，moof 跟 mdat 结对出现。moof 中包含了sample duration、sample size等信息，因此，fmp4 可以边生成边播放。
-   fmp4 中的 moov box 只存储文件级别的媒体信息，因此 moov box 的体积比传统的 MP4 中的 moov box 体积要小很多。
-   普通mp4下发整个.mp4文件，fmp4既可以下发完整 .mp4 文件，也可拆成分片文件（.m4s）和初始化文件（init.mp4）下发

# webm

WEBM 文件是以 WebM 格式保存的视频，这是一种开放的、免版税的格式，专为在网络上共享视频而设计。 WebM 使用类似于 Matroska (.MKV) 视频格式的容器结构，它同时存储音频和视频数据。视频使用 VP8 或 VP9 编解码器进行压缩，音频使用 Vorbis 或 Opus 音频编解码器进行压缩.

# 参考资料

[https://zhuanlan.zhihu.com/p/184577862](https://zhuanlan.zhihu.com/p/184577862)

[https://zhuanlan.zhihu.com/p/460011665](https://zhuanlan.zhihu.com/p/460011665)

[https://cloud.tencent.com/developer/article/1943082](https://cloud.tencent.com/developer/article/1943082)

[什么是MKV格式？和MP4什么区别？](https://zhuanlan.zhihu.com/p/147722579)

[vp8 vs vp9](https://webrtc.org.cn/vp8-vs-vp9/)