---
layout: post
title: "MessagePack 编解码"
description: "MessagePack 编解码"
categories: [io,netty]
tags: [io,netty]
redirect_from:
  - /2019/12/08/
---
# MessagePack
### 简介
> MessagePack 是一个搞笑的二进制序列化宽街概念，它像JSON一样支持不同语言的数据交换，但是他的性能更快，序列化之后的码流也更小，在业界得到了非常广泛的应用。MessagePack的特点如下：
> * 编解码搞笑，性能高。
> * 序列化之后的码流小。
> * 支持跨语言。

### 多语言支持
> 衡量序列化框架通用性的一个重要指标就是对多语言的支持，因为数据交换的双方很难保证必须使用一样的语言，如果序列化框架必须只能使用同一种语言，他就很难普及，例如JAVA的序列化。MessagePack提供了很多语言的支持，官方的列表如下：Java、Python、Ruby、Haskell、C#、OCnml、Lua、Go、C、C++等，可以参考官网：http://msgpack.org

### MessagePack Java API介绍

#### mavan种dependencies配置

``` xml
<dependencies>
  ...
  <dependency>
    <groupId>org.msgpack</groupId>
    <artifactId>msgpack</artifactId>
    <version>${msgpack.version}</version>
  </dependency>
  ...
</dependencies>
```
#### API编码解码代码

``` Java
// Create serialize objects.
List<String> src = new ArrayList<String>();
src.add("msgpack");
src.add("kumofs");
src.add("viver");

MessagePack msgpack = new MessagePack();
// Serialize
byte[] raw = msgpack.write(src);

// Deserialize directly using a template
List<String> dst1 = msgpack.read(raw, Templates.tList(Templates.TString));
System.out.println(dst1.get(0));
System.out.println(dst1.get(1));
System.out.println(dst1.get(2));

// Or, Deserialze to Value then convert type.
Value dynamic = msgpack.read(raw);
List<String> dst2 = new Converter(dynamic)
    .read(Templates.tList(Templates.TString));
System.out.println(dst2.get(0));
System.out.println(dst2.get(1));
System.out.println(dst2.get(2));
```

#### MessagePack开发包下载路径

> 你可以自己选择版本下载，网站地址：https://github.com/msgpack/msgpack-java/releases

# MessagePack编码器和解码器开发

### 准备

> 我们先准备好msgpack-0.6.6.jar与javassist-3.9.0.GA.jar两个jar包，可以去maven仓库或者官网下载。将准备好的两个jar包导入项目中。

### 编码器代码

``` Java
package io.netty.handler.coder.msgpack;

import org.msgpack.MessagePack;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;

public class MsgPackEncoder extends MessageToByteEncoder<Object>{

	@Override
	protected void encode(ChannelHandlerContext arg0, Object arg1, ByteBuf arg2) throws Exception {
		MessagePack msgPack = new MessagePack();
		byte[] raw = msgPack.write(arg1);
		arg2.writeBytes(raw);
	}
	
}

```

> 主要继承MessageToByteEncoder类，实现encode方法，将数据编码为byte数组。写入ByteBuf中。

### 解码器代码

``` Java
package io.netty.handler.coder.msgpack;

import java.util.List;

import org.msgpack.MessagePack;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToMessageDecoder;

public class MsgPackDecoder extends MessageToMessageDecoder<ByteBuf>{

	@Override
	protected void decode(ChannelHandlerContext arg0, ByteBuf arg1, List<Object> arg2) throws Exception {
		final byte[] array;
		final int length = arg1.readableBytes();
		array = new byte[length];
		arg1.getBytes(arg1.readerIndex(), array, 0, length);
		MessagePack msgPack = new MessagePack();
		arg2.add(msgPack.read(array));
	}

}

```

> 主要实现MessageToMessageDecoder类，将byte数组通过read方法反序列化为Object对象，然后将对象加入到解码列表arg2中。这就是MessagePack实现的编解码功能。 

### 粘包/半包支持

> 我们可以使用Netty提供的LengthFieldPrepender和LengthFieldBasedFrameDecoder来结合新开发的MessagePack编解码框架，来实现TCP粘包/半包的支持。使用方法跟之前介绍的使用方法一样，在initChannel方法中，将这两个类加入，MsgpackDecoder接收到的就永远是整包消息了。

##### 代码日下
``` Java
protected void initChannel(SocketChannel ch) throws Exception {
	//LengthFieldBasedFrameDecoder用于处理半包消息
	//这样后面的MsgpackDecoder接收的永远是整包消息
	ch.pipeline().addLast("frameDecoder",new LengthFieldBasedFrameDecoder(65535,0,2,0,2));
	ch.pipeline().addLast("msgpack decoder",new MsgPackDecoder());
	//在ByteBuf之前增加2个字节的消息长度字段
	ch.pipeline().addLast("frameEncoder",new LengthFieldPrepender(2)); 
	ch.pipeline().addLast("msgpack encoder",new MsgpackEncoder());
	ch.pipeline().addLast(new EchoClientHandler(sendNumber));
}
```

### 总结

> 本节主要讲了MessasgePack序列化框架，然后介绍了对TCP粘包和闭包的支持。后续章节，我们会对lengthFieldPrepender和LengthFieldBasedFrameDecoder进行详细讲解。