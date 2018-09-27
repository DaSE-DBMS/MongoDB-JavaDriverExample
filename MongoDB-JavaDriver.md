## prerequisite

### 1.[安装JDK](https://docs.oracle.com/javase/10/install/overview-jdk-10-and-jre-10-installation.htm#JSJIG-GUID-8677A77F-231A-40F7-98B9-1FD0B48C346A)

### 2.[安装Maven](http://maven.apache.org/)

## build
```
cd JavaDriverExample
mvn clean install
```

## run
```
java -jar target/JavaDriverExample-1.0-SNAPSHOT-jar-with-dependencies.jar
```

把pom.xml中的mainclass

```
                                <manifest>
                                    <mainClass>PojoQuickTour</mainClass>
                                </manifest>
```
换成其它含有main方法的类，比如QuickTour
rebuild
run
