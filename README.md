### 扫描组件文档 ###

## API 接口 ##
 //设置是否自动扫描 
`public void setIsAutoScan(boolean autoScan);`

 //设置扫描完毕是否回调是在主线程执行 默认在扫描线程 
`public void setCallBackInMainThread(boolean inMainThread);`

 //设置是否忽略nomedia 文件标志，如果忽略，则有.nomedia的目录也会被扫描 
`public void setIgnoreNomedia(boolean b);`

 //设置黑名单目录 黑名单内的目录一定不会被扫描 
public void setWhiteListDir(String[] whiteListDir);

 //设置扫描监听 扫描完后会回调listener 
public void addScanListener(LocalFileCacheManager.ScannerListener listener);

 //设置目录递归的最大深度 默认是25 
public void setMaxDirDepth(int depth);

 //获取新增的文件 将已有的文件map 传进去 会返回来去重后新增的文件 
public List<T> getNewEntities(HashMap<String, Boolean> existFiles);

 //获取扫描到的所有文件 
public List<T> getAllEntities();

 //启动扫描 
public void start();
 //停止扫描 不会立即停止 只会加快扫完 
public void stop();

 //设置debug 状态 debug为true时会有很多log打印出来 
`public void setDebug(boolean isDebug);`

 //设置是否忽略.(点）开头的目录（一般是隐藏目录 不会有媒体文件) 
`public void setIgnoreHiddenDir(boolean b);`

 //设置支持的文件类型 参数是后缀名list {".mp3",".m4a"} 
`public void setSupportedFileTypes(List<String> types);`

 //设置文件过滤器 在这里设置自定义的需要过滤的文件规则 类型FileFilter 
`public void setFileFilter(FilterUtil.IFileFilter fileFilter);`

 //设置目录过滤器 在这里设置自定义的需要过滤的目录规则 类型FileFilter 
`public void setDirFilter(FilterUtil.IDirFilter dirFilter);`

 //设置文件生成器 即定义扫描返回来的对象例如是一个还是[String|Video/Photo|Song|File]之类 
`public void setEntityGenerator(EntityGenerator<T> generator);`

## 调用DEMO ##
1.创建一个scanner(SongInfo 即是扫描完成后返回的item 的类型)
`FileScanner fileScanner = new FileScanner<SongInfo>(mContext);`

2.注册监听器

    fileScanner.addScanListener(new ScannerListener() { 
    @Override
    public void onScanBegin(boolean bFirst) {
        Log.d(TAG, "onScanBegin!!!!!!!!!!!!");
        //do sth here 
    }
    @Override
    public void onScanEnd(boolean needUpdate) {
        Log.d(TAG, "!!!!!!!!!!!onScanEnd!!!!!!!!!!!!");
        //do sth here 
    }
    });
3.启动扫描
`fileScanner.start();`

## 源代码文件结构 ##
* filescanner.so NDK层动态库
* Config.java 配置文件
* FileInfo.java 文件元数据抽象类
* DBHelper.java 数据库工具类
* FileScannerJava.java JAVA层扫描逻辑主类
* FileScannerJni.java NDK层扫描逻辑主类
* FilterUtil.java 扫描过滤规则工具类
* LocalFileCacheManager.java 数据库逻辑类
* ScannerUtils.java 工具类
* ScannerWrapper.java C++及JAVA层Wrapper

## 扫描组件实现方案 ##
* 为了兼容有些手机不支持NDK 增加了JAVA层 如果so 加载失败会调用


## 扫描策略 ##
1. 首次安装启动时进行一次全盘扫描，先扫描媒体库，再扫描磁盘文件，然后去重最后存入数据库，并展示界面.
2. 非首次安装启动时进行增量扫描，同样先扫描媒体库，再扫描有变更（修改时间）的目录，然后去重最后存入数据库，并展示界面.
3. 程序app从后台切换到前台时触发一次增量扫描
4. 收到媒体库监听时触发一次增量扫描
5. 用户手动扫描时清除之前数据进行一次全盘扫描
## 其它 ##

