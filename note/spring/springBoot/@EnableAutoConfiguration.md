### @EnableAutoConfiguration

最早接触的是基础的spring，为了引用二方包提供bean，还需要在xml中增加对应的包` `或者增加注解`@ComponentScan({ "xxx"})`。当时觉得挺urgly的，但也没有去研究有没有更好的方式。

直到接触Spring Boot 后，发现其可以自动引入二方包的bean。不过一直没有看这块的实现原理。直到最近面试的时候被问到。所以就看了下实现逻辑。

### 使用姿势

讲原理前先说下使用姿势。

在project A中定义一个bean。

```
package com.wangzhi;

import org.springframework.stereotype.Service;

@Service
public class Dog {
}
```

并在该project的`resources/META-INF/`下创建一个叫`spring.factories`的文件，该文件内容如下

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.wangzhi.Dog
```

然后在project B中引用project A的jar包。

projectA代码如下：

```
package com.wangzhi.springbootdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
public class SpringBootDemoApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringBootDemoApplication.class, args);
        System.out.println(context.getBean(com.wangzhi.Dog.class));
    }

}
```

打印结果：

```
com.wangzhi.Dog@3148f668
```

### 原理解析

总体分为两个部分：一是收集所有`spring.factories`中`EnableAutoConfiguration`相关bean的类，二是将得到的类注册到spring容器中。

#### 收集bean定义类

在spring容器启动时，会调用到`AutoConfigurationImportSelector#getAutoConfigurationEntry`

```
protected AutoConfigurationEntry getAutoConfigurationEntry(
        AutoConfigurationMetadata autoConfigurationMetadata,
        AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    // EnableAutoConfiguration注解的属性：exclude，excludeName等
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 得到所有的Configurations
    List<String> configurations = getCandidateConfigurations(annotationMetadata,
            attributes);
    // 去重
    configurations = removeDuplicates(configurations);
    // 删除掉exclude中指定的类
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = filter(configurations, autoConfigurationMetadata);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

`getCandidateConfigurations`会调用到方法`loadFactoryNames`：

```
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
        // factoryClassName为org.springframework.boot.autoconfigure.EnableAutoConfiguration
		String factoryClassName = factoryClass.getName();
        // 该方法返回的是所有spring.factories文件中key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的类路径
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}


public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
            // 找到所有的"META-INF/spring.factories"
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
                // 读取文件内容，properties类似于HashMap，包含了属性的key和value
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryClassName = ((String) entry.getKey()).trim();
                    // 属性文件中可以用','分割多个value
					for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryClassName, factoryName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

#### 注册到容器

在上面的流程中得到了所有在`spring.factories`中指定的bean的类路径，在`processGroupImports`方法中会以处理[@import](https://github.com/import)注解一样的逻辑将其导入进容器。

```
public void processGroupImports() {
    for (DeferredImportSelectorGrouping grouping : this.groupings.values()) {
        // getImports即上面得到的所有类路径的封装
        grouping.getImports().forEach(entry -> {
            ConfigurationClass configurationClass = this.configurationClasses.get(
                    entry.getMetadata());
            try {
                // 和处理@Import注解一样
                processImports(configurationClass, asSourceClass(configurationClass),
                        asSourceClasses(entry.getImportClassName()), false);
            }
            catch (BeanDefinitionStoreException ex) {
                throw ex;
            }
            catch (Throwable ex) {
                throw new BeanDefinitionStoreException(
                        "Failed to process import candidates for configuration class [" +
                                configurationClass.getMetadata().getClassName() + "]", ex);
            }
        });
    }
}

private void processImports(ConfigurationClass configClass, SourceClass currentSourceClass,
			Collection<SourceClass> importCandidates, boolean checkForCircularImports) {
	...
    // 遍历收集到的类路径
    for (SourceClass candidate : importCandidates) {
       ...
        //如果candidate是ImportSelector或ImportBeanDefinitionRegistrar类型其处理逻辑会不一样，这里不关注
     	// Candidate class not an ImportSelector or ImportBeanDefinitionRegistrar ->
						// process it as an @Configuration class
						this.importStack.registerImport(
								currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
		// 当作 @Configuration 处理			
        processConfigurationClass(candidate.asConfigClass(configClass));
   ...
}
            
    ...
}
```

可以看到，在第一步收集的bean类定义，最终会被以`Configuration`一样的处理方式注册到容器中。

### End

`@EnableAutoConfiguration`注解简化了导入了二方包bean的成本。提供一个二方包给其他应用使用，只需要在二方包里将对外暴露的bean定义在`spring.factories`中就好了。对于不需要的bean，可以在使用方用`@EnableAutoConfiguration`的`exclude`属性进行排除。

> 全文来自 https://github.com/farmerjohngit/myblog/issues/17