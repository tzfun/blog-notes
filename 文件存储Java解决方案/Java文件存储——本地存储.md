在我们平时的开发中难免会遇到一些有关文件存储的问题，这里根据我平时的开发经验，记录一下常用的的两大类文件存储方案，这里写上第一种**本地存储方案**，可能方法不是最优或者有什么不好之处，欢迎与我交流探讨。

PS：
* 以下解决方案都仅适用于单线程，多线程情况需要做一些优化处理，后期更新多线程解决方案。
* 代码演示环境基于Springboot
# 说明
一般在局域网或者自己搭建文件服务器时，都是将存储的文件存放到本地服务器或者局域网内的其他服务器，局域网内文件传输就不多介绍啦，一般用http、ftp协议等来做处理，这里我主要介绍一下网站开发中如何接收并处理前端传来的文件。
# 接收文件
这里我写了一个Controller层和Service层的，方便编码。

Controller层如下
```
/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 20:55 2018/11/16
 * @Description: 文件操作控制层
 */
@RestController
public class FileController {

    // 自动装配文件操作业务逻辑层
    @Autowired
    private FileService fileService;

    @PostMapping("/upload_file")
    public ResultFileVO uploadFile(@RequestParam("file") MultipartFile file){
        return fileService.uploadFile(file);
    }
}
```
该方法我们就已经将文件读到了后台，但是并没有写入磁盘，我们还需要做另外的配置，所以Service层的代码稍后贴出来。
# 静态文件路径配置
如果想要用户通过链接直接访问到传的文件（比如直接访问图片），我们就需要对其进行一个路径映射。

首先在application.yml文件下配置（这里省去了数据库的配置）
```
server:
  port: 80
  host: http://localhost:${server.port} # 服务器地址
spring:
  servlet:
    multipart:
      max-file-size: 3MB    # 文件最大限制
      max-request-size: 10MB   # 最大请求限制
  resources:    # 配置静态路径映射
    static-locations: file:${file.img.location},file:${file.resource.location}, classpath:/static/,classpath:/resources/
  mvc:
    static-path-pattern: /**
file:   # 自定义的文件存储路径（你的服务器磁盘地址）
  img:
    location: C:/Users/TZPlay/desktop/data-management-plaform/article-img # 图片文件存储在本地的目录
  resource:
    location: C:/Users/TZPlay/desktop/data-management-plaform/resource # 资源文件存储在本地的目录
```
然后我们需要将配置注入（虽然springboot会自动注入，但是不知道是版本问题还是什么，有时候不起作用，为了以防万一这里还是手动配置一下）
```
/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 23:30 2018/11/7
 * @Description:
 */
@Configuration
public class DefaultViewConfiguration extends WebMvcConfigurerAdapter {

    /**
     * 存放图片的路径
     */
    @Value("${file.img.location}")
    private String imgLocation;
    /**
     * 存放资源文件的路径
     */
    @Value("${file.resource.location")
    private String resourceLocation;

    /**
     * 配置默认首页
     * @param registry
     */
    @Override
    public void addViewControllers(ViewControllerRegistry registry){
        registry.addViewController("/").setViewName("forward:/index.html");
        registry.setOrder(Ordered.HIGHEST_PRECEDENCE);
        super.addViewControllers(registry);
    }

    @Bean
    public MultipartConfigElement multipartConfigElement(){
        MultipartConfigFactory factory = new MultipartConfigFactory();
        // 文件最大允许(方法已过时)
//        factory.setMaxFileSize("3MB");
//        factory.setMaxRequestSize("10MB");
        return factory.createMultipartConfig();
    }
}
```
# 存储文件
Service代码如下
```
/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 21:01 2018/11/16
 * @Description: 文件操作业务逻辑层接口类
 */
public interface FileService {

    ResultFileVO uploadFile(MultipartFile file);
}

```
```

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 21:01 2018/11/16
 * @Description: 文件处理业务逻辑层实现类
 */
@Service
public class FileServiceImp implements FileService {

    private static final Logger logger = LoggerFactory.getLogger(FileServiceImp.class);

    /**
     * 存放资源文件的路径
     */
    @Value("${file.resource.location}")
    private String resourceLocation;

    @Value("${server.host}")
    private String host;

    @Override
    public ResultFileVO uploadFile(MultipartFile file) {
        ResultFileVO resultFileVO = new ResultFileVO();
        try{
            if (file.isEmpty() || StringUtils.isBlank(file.getOriginalFilename())) {
                throw new Exception("File not empty!");
            }
            String contentType = file.getContentType();
            if(!contentType.contains("")){
                throw new Exception("File format error!");
            }
            String originalFilename = file.getOriginalFilename();
            logger.info("上传文件：name={},type={}",originalFilename,contentType);
            String fileName = saveFile(file,resourceLocation);
            String url = host + "/" + fileName;
            logger.info("保存文件：url={}",url);
            resultFileVO.setUrl(url);
            resultFileVO.setUploaded(true);
            resultFileVO.setMsg("上传成功");
            return resultFileVO;
        }catch (Exception e){
            e.printStackTrace();
            resultFileVO.setUrl("/");
            resultFileVO.setMsg("上传出错，"+e.getMessage());
            resultFileVO.setUploaded(false);
            return resultFileVO;
        }
    }
    /**
     * 文件存储
     * @param file MultipartFile类型
     * @param filePath 文件路径
     * @return boolean
     */
    public String saveFile(MultipartFile file, String filePath) throws IOException {
        File localFile = new File(filePath);
        if(!localFile.exists()){
            localFile.mkdirs();
        }
        FileInputStream fileInputStream = (FileInputStream) file.getInputStream();
        String fileName = new Date().getTime() + "." + file.getOriginalFilename().split("\\.")[1];
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(filePath + File.separator + fileName));
        byte[] bs = new byte[1024];
        int len;
        while((len = fileInputStream.read(bs)) !=-1 ){
            bos.write(bs, 0, len);
        }
        bos.flush();
        bos.close();
        return fileName;
    }
}

```
在项目中应该对方法进行一些分类处理，一般我是写在一个FileUtil类里面，这里为了演示就直接写出逻辑代码吧。
# 测试
这里我用Postman对文件存储进行测试
![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8/20181122173605.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8/20181122173605.jpg)
我们看到控制台打印得到日志消息
![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8/20181122173618.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8/20181122173618.jpg)
这个时候我们直接从浏览器输入这个网址就可以下载到文件了（如果浏览器支持该类型文件会进入浏览模式，比如图片或者ptf等等，如果是浏览器不支持的文件会自动下载）
![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8/20181122173720.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8/20181122173720.jpg)


本博文仅仅实现了本地文件存储的方案，在实际应用中还需要加一些分类管理、权限管理、文件处理等工作，应用的时候有所不同，这就看开发者如何去使用它了。