# apktool2.4.1
## 功能: 兼容不压缩文件过多(donotcompress)
### 失败原因:
_apk 在build 的时候存在不压缩文件过多,导致build 失败. 主要原因:_


     try {
                OS.exec(cmd.toArray(new String[0]));
                LOGGER.fine("aapt2 compile command ran: ");
                LOGGER.fine(cmd.toString());
            } catch (BrutException ex) {
                throw new AndrolibException(ex);
          }
   OS.exec(cmd.toArray(new String[0])); 里面cmd的大小 不超过8K, 所以当donotcompress 文件过多的时候就超过8K 导致build 失败.

### 优化思路:

_将不压缩文件名 存储在一个文件中, 而不是cmd中. 这样build 的时候 不压缩的文件都会被压缩.  由于不压缩文件名存储在一个文件中, build 成功之后可 依据这个
文件 进行重新压缩. 在这个文件中的文件名 压缩算法 设置 store, 不再文件中的保持原有的压缩算法和压缩等级_

### apktool 修改代码片段:
__文件路径 : apktool-lib\src\main\java\brut\androlib\res\AndrolibResources.java__

```` 
if (apkOptions.doNotCompress != null)
{
        try{
              String  nocompress_file = apkOptions.doNotCompress_file;
              File filename = new File(nocompress_file);
              if (nocompress_file != "")
              {
                  FileWriter fw = new FileWriter(nocompress_file);
                  for (String file : apkOptions.doNotCompress)
                  {
                      fw.write(file);
                      fw.write("\r\n");
                  }
                  fw.close();
              }

          }catch(IOException ex)
          {
              throw new AndrolibException(ex);
          }
}
````




### apktool 源码编译

1.  __git clone git://github.com/iBotPeaches/Apktool.git__

2.  __cd Apktool__

3.  __For steps 3-5 use ./gradlew for unix based systems or gradlew.bat for windows.__

4.  __[./gradlew][gradlew.bat] build shadowJar - Builds Apktool, including final binary.__

5.  __Optional (You may build a Proguard jar) [./gradlew][gradlew.bat] build shadowJar proguard__

    __After build completes you should have a jar file at: ./brut.apktool/apktool-cli/build/libs/apktool-xxxxx.jar__
    
### apktool 命令使用:
  ### decode:
  
  __这里将 -f 选项的参数 作为存储 不压缩文件名的文件(decode 的时候 不需要 可以任意一个字符串)__
  
  __apktool : apktool 路径__
  
  __tempdir : decode 路径__
  
  __"aliad" : decode 的时候这个任意写一个字符串(decode )__
  
  __"java -jar %s d  -o %s   -f %s" % (apktool,tempdir,self.framework,"aliad")__
  
      
  ### build:
  
  __apktool_jar_path: apktool的路径__
  
  __tempdir:        apktool 解包路径__
  
  __apkpath:        输出apk的路径__
  
  __DoNoCompress  : 存储不压缩文件名的 文件路径__
  
  "java -jar %s b %s -o %s  -f %s" % (apktool_jar_path, tempdir, apkpath, DoNoCompress)
  
   
  ### 微信: yangsenhehe 如果对你有所帮助, 记得点赞, star偶.
