## Day2

### 网络编程

*网络编程三要素：*

1. IP地址:IPv4
2. 端口号:1-65535(1-1024保留，一般用1024-50000)
3. 网络协议：TCP/UDP

### UDP

UDP服务器端使用`DatagramSocket`初始化要传入端口。使用`send`方法发送，数据需要使用DatagramPacket，而传入参数需要一个byte数组。通过`DatagramSocket`对象的`receive`接收。通过`DatagramSocket`的getData方法获取数据。

客户端接受：

```java
package com;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

/*
 * 使用UDP协议接收数据
		创建接收端Socket对象
		接收数据
		解析数据
		输出数据
		释放资源

 */
public class ReceiveDemo {
	public static void main(String[] args) throws IOException {
		//创建接收端Socket对象
		DatagramSocket ds = new DatagramSocket(8888);
		//接收数据
		//DatagramPacket(byte[] buf, int length) 
		byte[] bys = new byte[1024];
		DatagramPacket dp = new DatagramPacket(bys,bys.length);
				
		System.out.println(1);
		ds.receive(dp);//阻塞
		System.out.println(2);
		
		//解析数据
		//InetAddress getAddress() : 获取发送端的IP对象
		InetAddress address = dp.getAddress();
		//byte[] getData()  ：获取接收到的数据，也可以直接使用创建包对象时的数组
		byte[] data = dp.getData();
		//int getLength()  ：获取具体收到数据的长度
		int length = dp.getLength();
		
		//输出数据
		System.out.println("sender ---> " + address.getHostAddress());
		//System.out.println(new String(data,0,length));
		System.out.println(new String(bys,0,length));
		//释放资源
		ds.close();

	}
}
```

客户发送端

```java
package com;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;

/*
 * 使用UDP协议发送数据
		创建发送端Socket对象
		创建数据并打包
		发送数据
		释放资源
 * 
 * DatagramSocket:此类表示用来发送和接收数据,基于UDP协议的
 * 
 * DatagramSocket() ：创建Socket对象并随机分配端口号
 * DatagramSocket(int port) ：创建Socket对象并指定端口号
 */
public class SendDemo {
	public static void main(String[] args) throws IOException  {
		//创建发送端Socket对象
		DatagramSocket ds = new DatagramSocket();
		//创建数据并打包
		/*
		 * DatagramPacket :此类表示数据报包
		 * 数据 byte[]
		 * 设备的地址 ip
		 * 进程的地址  端口号
		   DatagramPacket(byte[] buf, int length, InetAddress address, int port) 
		 */
		
		String s = "hello udp,im comming!";
		byte[] bys = s.getBytes();
		int length = bys.length;
		InetAddress address = InetAddress.getByName("itheima");//发送给当前设备
		int port = 8888;
		//打包
		DatagramPacket dp = new DatagramPacket(bys,length,address,port);
		//发送数据
		ds.send(dp);
		//释放资源
		ds.close();
	}
}
```

### TCP

 TCP使用Socket对象和ServerSocket对象完成，需要的流由socket对象获取。

一个TCP的客户端

```java
package com.itheima_05;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.Socket;

/*
 	需求：使用TCP协议发送数据，并将接收到的数据转换成大写返回
 	
 	客户端发出数据
 	服务端接收数据
 	服务端转换数据
 	服务端发出数据
 	客户端接收数据
 	
 */
public class ClientDemo {
	public static void main(String[] args) throws IOException {
		//创建客户端Socket对象
		Socket s = new Socket(InetAddress.getByName("itheima"),10010);
		//获取输出流对象
		OutputStream os = s.getOutputStream();
		//发出数据
		os.write("tcp,i m comming again!!!".getBytes());

		
		//获取输入流对象
		InputStream is = s.getInputStream();
		byte[] bys = new byte[1024];
		int len;//用于存储读取到的字节个数
		//接收数据
		len = is.read(bys);
		//输出数据
		System.out.println(new String(bys,0,len));
		
		//释放资源
		s.close();
		
	}
}
```

一个TCP的服务端

```java
package com.itheima_05;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class ServerDemo {
	public static void main(String[] args) throws IOException {
		//创建服务端Socket对象
		ServerSocket ss = new ServerSocket(10010);
		//监听
		Socket s = ss.accept();
		//获取输入流对象
		InputStream is = s.getInputStream();
		//获取数据
		byte[] bys = new byte[1024];
		int len;//用于存储读取到的字节个数
		len = is.read(bys);
		String str = new String(bys,0,len);
		//输出数据
		System.out.println(str);
		//转换数据
		String upperStr = str.toUpperCase();
		//获取输出流对象
		OutputStream os = s.getOutputStream();
		//返回数据（发出数据）
		os.write(upperStr.getBytes());
		
		//释放资源
		s.close();
		//ss.close();//服务端监听一般不关闭
	}
}
```



### NIO（New IO）

##### Buffer

##### Channel

##### SocketChannel

##### ServerSokcetChannel

##### DatagramChannel

见补充-->NIO