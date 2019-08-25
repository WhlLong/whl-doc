```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	//排除特定的配置
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	//排除特定的配置
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	//根据包路径扫描
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	//根据class类扫描
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

}
```



SpringBoot自动配置的核心在于SpringBootApplication注解，SpringBootApplication注解是一个组合注解，其中最重要的注解有三个：

1. @SpringBootConfiguration
2. @ComponentScan(excludeFilters = {
   		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
   		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
3. @EnableAutoConfiguration



**@SpringBootConfiguration**其实就是一个@Configuration注解

**@ComponentScan**的功能其实就是自动扫描并加载符合条件的组件或bean定义，最终将这些bean定义加载到容器中

**@EnableAutoConfiguration**的作用是开启自动配置功能，其原理是借用@Import的帮助，将所有符合自动配置条件的bean定义加载到ioc容器中。





```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};

	String[] excludeName() default {};

}

```





根据原来可以看出@EnableAutoConfiguration注解借用@Import注解加载了一个名为AutoConfigurationImportSelector的组件。

AutoConfigurationImportSelector中有一个覆盖了父类的方法selectImports，这个方法加载了所有的候选配置，

去重以后再将已经被排除的配置给剔除出去。

```java
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
        //获取所有的候选配置
		List<String> configurations = getCandidateConfigurations(annotationMetadata,
				attributes);
        //移除已有的配置
		configurations = removeDuplicates(configurations);
        //移除已经配置了需要排除的配置
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = filter(configurations, autoConfigurationMetadata);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return StringUtils.toStringArray(configurations);
	}
```





再来看看它是如何加载所有的候选配置的：

```java
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```



通过getCandidateconfigurations方法可以看出加载候选配置是通过SpringFactoriesLoader类来实现的：

```java
	
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

	public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}

	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					List<String> factoryClassNames = Arrays.asList(
							StringUtils.commaDelimitedListToStringArray((String) entry.getValue()));
					result.addAll((String) entry.getKey(), factoryClassNames);
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



SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置,即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类,这些配置类最终会通过反射的形式实例化为标注了@Configuration的JavaConfig形式的IOC容器配置类。这些配置类会根据内部的配置去classpath中寻找该类的依赖类。需要注意的是，生成功能类的原则是自定义优先，只有在没有额外自定义时才会使用自定义装配类。



以RedisAutoConfiguration为例：

```java
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	public StringRedisTemplate stringRedisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```



可以看出它通过@Bean注解将RedisTemplate和StringRedisTemplate实例加入到了IOC容器中。

这里有几个注解需要解释一下：

@ConditionalOnClass： 这是个条件注解，仅当classpath中可以找到指定的类时，该配置文件才能生效。

@EnableConfigurationProperties：开启默认熟悉的配置，该注解会自动加载配置文件中的配置，然后赋值到指定的属性配置类中。

@ConditionalOnMissingBean：仅当用户没有自定义的时候默认配置才会生效，当用户在配置文件中自定义时候就会覆盖默认的配置