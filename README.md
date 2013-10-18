# 使用keytool 和 openssl

服务端证书的一般工作流程如下：

1. 生成服务端的私钥 
2. 通过私钥生成证书请求文件
3. 生成CA证书文件
4. 通过CA证书文件对证书请求文件进行签名生成证书文件(.crt)

下面演示在这个过程中 openssl和keytool的一些使用方法

## 准备工作
建立自签名CA中心
	
	mkdir -p ./demoCA/{private,newcerts}
	touch ./demoCA/index.txt
	echo 01 > ./demoCA/serial

## 使用openssl 生成证书
1. 生成服务端私钥

	openssl genrsa -des3 -out server.key 1024

2. 通过该私钥生成证书请求文件
 
	openssl req -new -key server.key -out server.csr
	
3. 生成自签名CA证书

	openssl req -new -x509 -keyout ca.key -out ca.crt -config /System/Library/OpenSSL/openssl.cnf	
	
4. 通过CA自签名证书对证书请求文件进行签名，生成证书

	openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -notext -config /System/Library/OpenSSL/openssl.cnf
	
## 使用java keytool 生成证书 

1. 生成私钥及keystore 文件（私钥存放在keystore文件内)

	keytool -genkey -alias my_server -validity 365 -keyalg RSA -keysize 1024 -keypass 123456  -storepass 123456 -keystore server_keystore 
	
注：接下来会生成证书请求文件，并且使用刚才生成的CA证书对证书请求文件签名，所以需要保证填写的信息和生成CA证书的信息一致才能签名成功	

2.  生成证书请求文件

	keytool -certreq -alias my_server -sigalg MD5withRSA -file my_server.csr -keypass 123456 -storepass 123456 -keystore server_keystore 
	
3.  使用 ca 秘钥对证书请求文件进行签名
 
	openssl ca -in my_server.csr -out my_server.crt -cert ca.crt -keyfile ca.key -notext -config /System/Library/OpenSSL/openssl.cnf
	
4.  导入信任ca 根证书到keystroe

	 keytool -import -v -trustcacerts  -alias my_ca_root -file ca.crt -storepass 123456 -keystore server_keystore
	 
5. 把CA签名后的server端证书导入 keystore

	keytool -import -v -alias my_server -file my_server.crt -storepass 123456 -keystore server_keystore
	
### 其他操作 
此时应该可以看到keystore里有两个条目，根证书 my_ca_root和 tomcat_server.csr 的证书链长度为2

	keytool -list -v -keystore server_keystore 
	
删除存在的证书	

	keytool -delete -alias tomcat_server  -storepass 123456 -keystore server_keystore

追加证书: 如果keystoe文件已经存在可以在这个文件增加新的私钥

	keytool -genkey -alias twitter_fake  -validity 365 -keyalg RSA -keysize 1024 -keypass 123456  -storepass 123456 -keystore server_keystore 
	


## 使用openssl 生成多域名证书 


## 参考资料

http://zhouzhk.iteye.com/blog/136943

http://apetec.com/support/GenerateSAN-CSR.htm
