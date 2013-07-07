# 使用keytool 和 openssl

## 准备工作
建立自签名CA中心
	
	mkdir -p ./demoCA/{private,newcerts}
	touch ./demoCA/index.txt
	echo 01 > ./demoCA/serial

## 使用openssl 生成自签名证书 
	openssl req -new -x509 -keyout ca.key -out ca.crt -config /System/Library/OpenSSL/openssl.cnf	


## 生成java keystore文件

	keytool -genkey -alias tomcat_server -validity 365 -keyalg RSA -keysize 1024 -keypass 123456  -storepass 123456 -keystore server_keystore	

注：接下来会生成证书请求文件，并且使用刚才生成的CA证书对证书请求文件签名，所以需要保证填写的信息和生成CA证书的信息一致才能签名成功	
## 生成证书请求文件
	keytool -certreq -alias tomcat_server -sigalg MD5withRSA -file tomcat_server.csr -keypass 123456 -storepass 123456 -keystore server_keystore 
	
## 使用 ca 秘钥对证书请求文件进行签名
	openssl ca -in tomcat_server.csr -out tomcat_server.crt -cert ca.crt -keyfile ca.key -notext -config /System/Library/OpenSSL/openssl.cnf
	
## 导入信任ca 根证书到keystroe

	 keytool -import -v -trustcacerts  -alias my_ca_root -file ca.crt -storepass 123456 -keystore server_keystore
	 
## 把CA签名后的server端证书导入 keystore

	keytool -import -v -alias tomcat_server -file tomcat_server.crt -storepass 123456 -keystore server_keystore
	
## 查看keystore的证书文件
此时应该可以看到keystore里有两个条目，根证书 my_ca_root和 tomcat_server.csr 的证书链长度为2

	keytool -list -v -keystore server_keystore 
	
删除存在的证书	
	keytool -delete -alias tomcat_server  -storepass 123456 -keystore server_keystore
## 参考资料

http://zhouzhk.iteye.com/blog/136943
