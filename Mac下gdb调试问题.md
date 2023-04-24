利用VScode进行GDB调试时，出现see taskgated(8)异常，报错如下：

报错描述：

```
ERROR: Unable to start debugging. Unexpected GDB output from command "-exec-run". Unable to find Mach task port for process-id 29694: (os/kern) failure (0x5).  (please check gdb is codesigned - see taskgated(8))
```

证书名称：gdb_cert(证书名称自定义，根据个人习惯来取名)

#### 对GDB创建签名证书

1.创建签名证书。利用脚本自动化创建，脚本内容如下：

```
#!/bin/bash

# This script is copied from https://github.com/llvm/llvm-project/blob/main/lldb/scripts/macos-setup-codesign.sh

CERT="gdb_cert"

function error() {
    echo error: "$@"
    exit 1
}

function cleanup {
    # Remove generated files
    rm -f "$TMPDIR/$CERT.tmpl" "$TMPDIR/$CERT.cer" "$TMPDIR/$CERT.key" > /dev/null 2>&1
}

trap cleanup EXIT

# Check if the certificate is already present in the system keychain
security find-certificate -Z -p -c "$CERT" /Library/Keychains/System.keychain > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo Certificate has already been generated and installed
    exit 0
fi

# Create the certificate template
cat <<EOF >$TMPDIR/$CERT.tmpl
[ req ]
default_bits       = 2048        # RSA key size
encrypt_key        = no          # Protect private key
default_md         = sha512      # MD to use
prompt             = no          # Prompt for DN
distinguished_name = codesign_dn # DN template
[ codesign_dn ]
commonName         = "$CERT"
[ codesign_reqext ]
keyUsage           = critical,digitalSignature
extendedKeyUsage   = critical,codeSigning
EOF

echo Generating and installing lldb_codesign certificate

# Generate a new certificate
openssl req -new -newkey rsa:2048 -x509 -days 3650 -nodes -config "$TMPDIR/$CERT.tmpl" -extensions codesign_reqext -batch -out "$TMPDIR/$CERT.cer" -keyout "$TMPDIR/$CERT.key" > /dev/null 2>&1
[ $? -eq 0 ] || error Something went wrong when generating the certificate

# Install the certificate in the system keychain
sudo security add-trusted-cert -d -r trustRoot -p codeSign -k /Library/Keychains/System.keychain "$TMPDIR/$CERT.cer" > /dev/null 2>&1
[ $? -eq 0 ] || error Something went wrong when installing the certificate

# Install the key for the certificate in the system keychain
sudo security import "$TMPDIR/$CERT.key" -A -k /Library/Keychains/System.keychain > /dev/null 2>&1
[ $? -eq 0 ] || error Something went wrong when installing the key

# Kill task_for_pid access control daemon
sudo pkill -f /usr/libexec/taskgated > /dev/null 2>&1

# Exit indicating the certificate is now generated and installed
exit 0
```

2. 创建gdb.xml（文件名可以自定义）文件，内容如下：

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
   "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
       <key>com.apple.security.cs.allow-jit</key>
       <true/>
       <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
       <true/>
       <key>com.apple.security.cs.allow-dyld-environment-variables</key>
       <true/>
       <key>com.apple.security.cs.disable-library-validation</key>
       <true/>
       <key>com.apple.security.cs.disable-executable-page-protection</key>
       <true/>
       <key>com.apple.security.cs.debugger</key>
       <true/>
       <key>com.apple.security.get-task-allow</key>
       <true/>
   </dict>
   </plist>
   ```

3. 对gdb进行签名

   ```
   sudo codesign --entitlements gdb.xml -fs gdb_cert /usr/local/bin/gdb
   
   //或
   sudo codesign --entitlements gdb.xml -fs gdb_cert $(which gdb)
   ```

4. 重启系统或通过重启taskgated服务刷新证书和签名数据。

   ```
   sudo killall taskgated
   ```



常用命令

```
//查看创建的签名证书信息
security find-certificate -Z -p -c "gdb_cert" /Library/Keychains/System.keychain

//删除签名证书，-Z参数表示证书SHA-1 hash的值
sudo security delete-certificate -Z CC6905AEF2E9A5E0AD63A9C2B08B69D559E223A3 /Library/Keychains/System.keychain

 
```



