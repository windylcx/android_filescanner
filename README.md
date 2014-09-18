### 扫描组件文档 ###

## API 接口 ##
 //设置是否自动扫描 
`public void setIsAutoScan(boolean autoScan);`

 //设置扫描完毕是否回调是在主线程执行 默认在扫描线程 
`public void setCallBackInMainThread(boolean inMainThread);`

 //设置是否忽略nomedia 文件标志，如果忽略，则有.nomedia的目录也会被扫描 
`public void setIgnoreNomedia(boolean b);`

 //设置黑名单目录 黑名单内的目录一定不会被扫描 
`public void setWhiteListDir(String[] whiteListDir);`

 //设置扫描监听 扫描完后会回调listener 
`public void addScanListener(LocalFileCacheManager.ScannerListener listener);`

 //设置目录递归的最大深度 默认是25 
`public void setMaxDirDepth(int depth);`

 //获取新增的文件 将已有的文件map 传进去 会返回来去重后新增的文件 
`public List<T> getNewEntities(HashMap<String, Boolean> existFiles);`

 //获取扫描到的所有文件 
`public List<T> getAllEntities();`

 //启动扫描 
`public void start();`
 //停止扫描 不会立即停止 只会加快扫完 
`public void stop();`

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

#### 主体逻辑 ####
1. 扫描所有目录并写入数据库
2. 扫描所有文件并写入数据库
3. 扫描完毕触发回调，在回调中通过查询接口取得所有扫描的结果(文件列表，目录列表，新增文件等)

#### 扫描目录 ####
1. 递归扫描所有SDCard 及目录
2. 在扫描过程中判断该目录是否是黑名单目录，是则跳过继续扫描下一目录
3. 在扫描过程中判断该目录是否是白名单目录，是则忽略其它过滤条件（保证白名单的目录一定会被扫描)
4. 扫描过程中判断IgnoreHiddenDir 标志是否设置，如果设置了忽略则扫描隐藏目录（.开头),否则跳过该目录(默认跳过)
5. 扫描过程中判断IgnoreNomedia 标志是否设置，如果设置了忽略则扫描包含了.nomedia的目录,否则跳过该目录(默认跳过)
6. 扫描过程中通过路径path生成FileInfo(包含path,lastModified,count,type等),增加到一个List中
7. 所有目录扫描完毕之后通过JNI返回List<FileInfo> ，插入到数据库的目录表中，目录扫描完毕

#### 扫描文件 ####
1. 遍历所有目录，扫描该目录下的文件。
2. 扫描过程中判断文件是否是指定的类型，如果不是则跳过该文件
3. 扫描过程中通过path生成FileInfo(包含path,lastModified),增加到一个文件List中
4. 该目录扫描完毕，通过JNI返回List<FileInfo> ，插入到数据库的文件表中
5. 继续扫描下一目录直至所有目录扫描完毕.

#### 增量扫描的实现 ####
* 目录扫描完毕后会存到数据库里面 下次扫描时遍历所有目录的修改时间和数据库保存的修改时间是否一致，如果比保存的时间要新，则证明该目录有变化，将其加入变化的列表
* 对有变化的目录执行一遍文件扫描的逻辑，这样就实现了增量扫描。
* 扫描完毕更新目录的修改时间 写回数据库

#### 降级处理 ####
* 组件加载的时候会判断so是否加载成功 成功则扫描时用JNI组件去扫如果失败则用Java层接口扫描
* JNI层的接口和java层的接口完全一样

## 扫描策略 ##
1. 首次安装启动时进行一次全盘扫描，先扫描媒体库，再扫描磁盘文件，然后去重最后存入数据库，并展示界面.
2. 非首次安装启动时进行增量扫描，同样先扫描媒体库，再扫描有变更（修改时间）的目录，然后去重最后存入数据库，并展示界面.
3. 程序app从后台切换到前台时触发一次增量扫描
4. 收到媒体库监听时触发一次增量扫描
5. 用户手动扫描时清除之前数据进行一次全盘扫描

## 其它 ##

