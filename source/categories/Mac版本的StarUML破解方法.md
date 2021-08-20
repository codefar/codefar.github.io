# Mac版本的StarUML破解方法

StarUML是一款优秀的跨平台的UML工具。

1. 打开StarUML的安装包位置，修改配置文件。配置文件位置为

   ```
   /Applications/StarUML.app/Contents/www/license/node/LicenseManagerDomain.js
   ```



2. 修改验证。找下面这段代码实现

   ```
   function validate(PK, name, product, licenseKey) {
           var pk, decrypted;
           try {
               pk = new NodeRSA(PK);
               decrypted = pk.decrypt(licenseKey, 'utf8');
           } catch (err) {
               return false;
           }
           var terms = decrypted.trim().split("\n");
           if (terms[0] === name && terms[1] === product) {
               return {
                   name: name,
                   product: product,
                   licenseType: terms[2],
                   quantity: terms[3],
                   licenseKey: licenseKey
               };
           } else {
               return false;
           }
       }
   ```

修改为

```
function validate(PK, name, product, licenseKey) {
	return {
                  name: "你的名字", 
                  product: "StarUML",
                  licenseType: "vip",
                  quantity: "vip.com",
                  licenseKey: "你的license"
    };
}
```

**记住你填写的name和licenseKey（随便填写这两个字段）。**

3. 打开StarUML的

   help  —> enter license

   Name:你的名字
   licenseKey:你的license （你在上一步填写的信息）

   保存，会提示你错误，忽略。

4. 重启StarUML。

   ​

5. 