## gpg --help                                                                                                         ─╯
gpg (GnuPG) 2.2.41
libgcrypt 1.10.2-unknown
Copyright (C) 2022 g10 Code GmbH
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/zen/.gnupg
支持的算法：
公钥： RSA, ELG, DSA, ECDH, ECDSA, EDDSA
密文： IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
    CAMELLIA128, CAMELLIA192, CAMELLIA256
散列： SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
压缩：  不压缩, ZIP, ZLIB, BZIP2

语法：gpg [选项] [文件]
签名、检查、加密或解密
默认的操作依输入数据而定

### 命令：
```bash
 
 -s, --sign                         生成一份签名
     --clear-sign                   生成一份明文签名
 -b, --detach-sign                  生成一份分离的签名
 -e, --encrypt                      加密数据
 -c, --symmetric                    仅使用对称密文加密
 -d, --decrypt                      解密数据（默认）
     --verify                       验证签名
 -k, --list-keys                    列出密钥
     --list-signatures              列出密钥和签名
     --check-signatures             列出并检查密钥签名
     --fingerprint                  列出密钥和指纹
 -K, --list-secret-keys             列出私钥
     --generate-key                 生成一个新的密钥对
     --quick-generate-key           快速生成一个新的密钥对
     --quick-add-uid                快速添加一个新的用户标识
     --quick-revoke-uid             快速吊销一个用户标识
     --quick-set-expire             快速设置一个过期日期
     --full-generate-key            完整功能的密钥对生成
     --generate-revocation          生成一份吊销证书
     --delete-keys                  从公钥钥匙环里删除密钥
     --delete-secret-keys           从私钥钥匙环里删除密钥
     --quick-sign-key               快速签名一个密钥
     --quick-lsign-key              快速本地签名一个密钥
     --quick-revoke-sig             快速吊销一个密钥签名
     --sign-key                     签名一个密钥
     --lsign-key                    本地签名一个密钥
     --edit-key                     签名或编辑一个密钥
     --change-passphrase            更改密码
     --export                       导出密钥
     --send-keys                    将密钥导出到一个公钥服务器上
     --receive-keys                 从公钥服务器上导入密钥
     --search-keys                  在公钥服务器上搜索密钥
     --refresh-keys                 从公钥服务器更新所有密钥
     --import                       导入/合并密钥
     --card-status                  打印卡片状态
     --edit-card                    更改卡片上的数据
     --change-pin                   更改卡片的 PIN
     --update-trustdb               更新信任数据库
     --print-md                     打印消息摘要
     --server                       以服务器模式运行
     --tofu-policy VALUE            设置一个密钥的 TOFU 政策
``` 
## 控制诊断输出的选项:
```bash
 -v, --verbose                      详细模式
 -q, --quiet                        尽量减少提示信息
     --options FILE                 从 FILE 中读取选项
     --log-file FILE                将服务器模式的日志写入到 FILE
``` 
## 控制配置的选项:
```bash
     --default-key NAME             使用 NAME 作为默认的私钥
     --encrypt-to NAME              同时给以 NAME 为名称的用户标识加密
     --group SPEC                   设置电子邮件别名
     --openpgp                      使用严格的 OpenPGP 行为
 -n, --dry-run                      不做任何更改
 -i, --interactive                  覆盖前提示
```
## 输出控制选项:
```bash
 -a, --armor                        创建 ASCII 字符封装的输出
 -o, --output FILE                  写输出到 FILE
     --textmode                     使用规范的文本模式
 -z N                               设置压缩等级为 N （0 为禁用）
```
## 密钥导入导出控制选项:
```bash
     --auto-key-locate MECHANISMS   通过邮件地址定位密钥时使用机制 MECHANISMS
     --auto-key-import              从签名导入缺少的密钥
     --include-key-block            在签名中包含公钥
     --disable-dirmngr              禁用对 dirmngr 的所有访问
```
## 指定密钥选项:
```bash
 -r, --recipient USER-ID            为 USER-ID 加密
 -u, --local-user USER-ID           使用 USER-ID 来签名或者解密
```
（请参考手册页以获得所有命令和选项的完整列表）

例子：

 -se -r Bob [文件]          为用户 Bob 签名和加密
 --clear-sign [文件]        创建一个明文签名
 --detach-sign [文件]       创建一个分离签名
 --list-keys [名字]        列出密钥
 --fingerprint [名字]      显示指纹

请向 <https://bugs.gnupg.org> 报告程序缺陷。
请向 <i18n-zh@googlegroups.com> 邮件列表反映简体中文的翻译问题或建议。

