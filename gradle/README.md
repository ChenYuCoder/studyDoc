# maven and gradle
0. pom search
* http://mvnrepository.com/
* http://maven.aliyun.com/nexus

1. maven安装：http://www.yiibai.com/maven/maven_environment_setup.html

2. 创建Maven项目：http://blog.csdn.net/chuyuqing/article/details/28879477

3. 配置阿里云中心库：http://www.cnblogs.com/geektown/p/5705405.html
```
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror> 
```

4. 配置远程仓库：http://www.yiibai.com/maven/how-do-download-from-remote-repository-maven.html
```
 <repositories>
	<repository>
	    <id>java.net</id>
	    <url>https://maven.java.net/content/repositories/public/</url>
	</repository>
    </repositories>
```

5. 比较好的maven中央存储库
http://mvnrepository.com/
http://search.maven.org/
http://repository.sonatype.org/content/groups/public/
http://people.apache.org/repo/m2-snapshot-repository/
http://people.apache.org/repo/m2-incubating-repository/

6. Jar查询流程：
6.1. 在 Maven 本地资源库中搜索，如果没有找到，进入第 2 步，否则退出。
6.2. 在 Maven 中央存储库搜索，如果没有找到，进入第 3 步，否则退出。
6.3. 在java.net Maven的远程存储库搜索，如果没有找到，提示错误信息，否则退出


7. maven私服上传jar包  
* 普通列表项目 本地安装maven,并配置环境变量
*  需要修改maven中的conf\settings.xml 在<servers>节点中添加  
    <server>
    <id>maven-releases</id>
    <username>admin</username>
    <password>admin123</password>
    </server>
```
  *  使用命令行   示例如下: 将SoucketInvoke上传到maven-releases仓库，
  * Group为com.fairlink,
  * Name为SocketInvoke,
  * Version为1.0.0.0-releases
mvn deploy:deploy-file -DgroupId=com.fairlink -DartifactId=SocketInvoke -Dversion=1.0.0.0-releases -Dpackaging=jar -Dfile=F:\varlib\SocketInvoke.jar -Durl=http://192.168.10.253:8081/repository/maven-releases/ -DrepositoryId=maven-releases -DgeneratePom=true
```

8. Gradle学习：
http://blog.csdn.net/cai_iac/article/details/53689500

8.1 gradle 优点：
比ant ：增加依赖管理
比maven：增加编译灵活性
比他们俩：不采用xml配置格式，有底层语言

```
task test << {  //=doLast
    println "Hello World"
}

task test << {
    doFirst{    //执行前运行
        println "Hello World"
    }
    doLast{    //执行后运行
        println "Hello World"
    }
}

task testTest0 << {}    // gradle tT0 驼峰式命名采用缩写即可调用

task test2(dependsOn : test)    //task依赖，运行test2之前执行test

task test2(dependsOn : [t,t,t....]])  

task test3(type:Copy,dependsOn:t)   //type+dependsOn

gradle tasks //列出所有的task

-x //排除某task gradle test3 -x test2

//仓库  依序寻找
// mavenCentral()别名，表示依赖是从Central Maven 2 仓库中获取的。
// jcenter()别名，表示依赖是从Bintary’s JCenter Maven 仓库中获取的。
// mavenLocal()别名，表示依赖是从本地的Maven仓库中获取的。
repositories {
    flatDir {        //本地
        dirs 'lib'
    }
    aliyun {
        url "http://maven.aliyun.com/nexus/content/groups/public"
    }
    maven { url 'http://localhost:8081/nexus/content/repositories/releases/' }  //内网私有仓库
    mavenCentral()
    mavenLocal()
}

/**几种dependencies 方式
    依赖jar包
    compile：该依赖对于编译发行是必须的。
    runtime：该依赖对于运行时是必须的，默认包含编译时依赖。
    testCompile：该依赖对于编译测试是必须的，默认包含编译产品依赖和编译时依赖。
    testRuntime：该依赖对于测试是必须的，默认包含编译、运行时、测试编译依赖
*/
dependencies {
    compile (
        [group: 'foo', name: 'foo', version: '0.1'],
        [group: 'bar', name: 'bar', version: '0.1']
    )
}

dependencies {
    compile 'foo:foo:0.1', 'bar:bar:0.1'
}

dependencies {
    compile group: "foo", name: "foo", version: "0.1"
    testCompile group: "test", name: "test", version: "0.1"
    compile fileTree(dir: 'libs', include: '*.jar')     //依赖本地jar包
}

//源码路径设置
sourceSets {  
    main {  
        java {  
            srcDir 'src/main'  
        }  
    }
    test.java.srcDir 'src/test'
}

//jar编译设置
jar {
    //将编译需要的jar包编译到result.jar中
    from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } } 
    //设置主类
    manifest {
        attributes 'Main-Class': 'com.Main'
    }
}

//设置任务编码
tasks.withType(JavaCompile) {  
    options.encoding = "UTF-8"  
}

//默认属性  http://www.cnblogs.com/langtianya/p/5209515.html
sourceCompatibility = 1.8   // 设置 JDK 版本
version = '1.0'     //设置版本
group = "net.paoding"   //设置组名
artifacts  ="test"      //设置项目名
buildDir   ="build"     //编译路径，默认是build
project=""        //Project本身
name=""           //Project的名字
path=""           //Project的绝对路径
description=""    //Project的描述信息

//自定义属性
ext{
    buildZipDir = "build/zip"
    findbugsHome = System.getenv()['FINDBUGS_HOME'] //从系统变量中获取路径
}

//发布到本地maven http://blog.csdn.net/itbailei/article/details/47952819
repositories {
    mavenRepo url: 'file://' + new File(System.getProperty('user.home'), '.m2/repository').absolutePath
    mavenCentral()
}

uploadArchives {
    repositories {
        mavenDeployer {
            mavenLocal()
        }
    }
}

//发布到私库与本地路径 https://www.oschina.net/question/2559126_2214480
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url:"http://asdasdad/"){
                authentication(userName:"admin",password:"*******")
            }
            //以下三项可选
            pom.groupId = "groupId"
            pom.artifactId = "name"
            pom.version = "0.0.0"
        }
    }
}

uploadArchives {
    repositories {
        flatDir {
            dirs "c:/"
        }
    }
}

jar {
    manifest.attributes "Implementation-Title": "paoding-rose", "Implementation-Version": version
}
//删除文件
task cleanBuild () {
    delete "buildDir"
}
//创建文件
task mkBuild () {
    mkdir "buildDir"
}
//复制文件
task copy(type: Copy) {
    from "config"
    from fileTree( dir: libMainDir, include: ['*.jar'])
    from "config"
    into "${buildDir}/config/"
}
//findbugs  需要使用FindBugs插件
task findbugs(type: FindBugs) {
    findbugs {
        ignoreFailures = true
        effort ="max"
        classes = fileTree('build/classes/main')
        source = fileTree('src/main/')
    }

    tasks.withType(FindBugs){
        reports{
            xml.enabled = false
            html.enabled = true
        }
    }
    println "findbugs"
}
//checkstyle 需要使用Checkstyle插件
task checkstyle(type: Checkstyle) {
    checkstyle {
        // classes = fileTree('build/classes/main')
        // configProperties.checkstyleSuppressionsPath = file("${project.projectDirr}/config/quality/suppressions.xml").absolutePath 
        source = fileTree('src/main/')
        configFile = new File("../docs/checkstyle/checkstyle.xml")
        ignoreFailures = true
        showViolations = false
    }

    println "checkstyle"
}

//插件使用
apply plugin: 'java'
apply plugin: 'findbugs'
apply plugin: 'checkstyle'
apply plugin: 'war'         //webAppDirName 指定webapp路径
apply plugin: 'maven'
```
   