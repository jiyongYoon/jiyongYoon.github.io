---
title: "NoClassDefFoundError"
layout: single
categories: java
typora-root-url: ..\..\images\
toc: true
tag: [java_error]
---



# NoClassDefFoundError

- Test코드 작성 후 Test를 돌리고 있는데 해당 오류가 발생함.

  `java.lang.NoClassDefFoundError: org/springframework/plugin/metadata/SimplePluginMetadata`



## - Error Log

```
org/springframework/plugin/metadata/SimplePluginMetadata
java.lang.NoClassDefFoundError: org/springframework/plugin/metadata/SimplePluginMetadata
	at java.base/java.lang.ClassLoader.defineClass1(Native Method)
	at java.base/java.lang.ClassLoader.defineClass(ClassLoader.java:1016)
	at java.base/java.security.SecureClassLoader.defineClass(SecureClassLoader.java:174)
	at java.base/jdk.internal.loader.BuiltinClassLoader.defineClass(BuiltinClassLoader.java:801)
	at java.base/jdk.internal.loader.BuiltinClassLoader.findClassOnClassPathOrNull(BuiltinClassLoader.java:699)
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClassOrNull(BuiltinClassLoader.java:622)
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:580)
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521)
	at java.base/java.lang.Class.forName0(Native Method)
	at java.base/java.lang.Class.forName(Class.java:398)
	at java.base/sun.reflect.generics.factory.CoreReflectionFactory.makeNamedType(CoreReflectionFactory.java:114)
	at java.base/sun.reflect.generics.visitor.Reifier.visitClassTypeSignature(Reifier.java:125)
	at java.base/sun.reflect.generics.tree.ClassTypeSignature.accept(ClassTypeSignature.java:49)
	at java.base/sun.reflect.generics.visitor.Reifier.reifyTypeArguments(Reifier.java:68)
	at java.base/sun.reflect.generics.visitor.Reifier.visitClassTypeSignature(Reifier.java:138)
	at java.base/sun.reflect.generics.tree.ClassTypeSignature.accept(ClassTypeSignature.java:49)
	at java.base/sun.reflect.generics.repository.ClassRepository.computeSuperInterfaces(ClassRepository.java:117)
	at java.base/sun.reflect.generics.repository.ClassRepository.getSuperInterfaces(ClassRepository.java:95)
	at java.base/java.lang.Class.getGenericInterfaces(Class.java:1138)
	at org.springframework.core.ResolvableType.getInterfaces(ResolvableType.java:501)
	at org.springframework.core.ResolvableType.as(ResolvableType.java:449)
	at org.springframework.core.ResolvableType.forClass(ResolvableType.java:1055)
	at org.springframework.plugin.core.config.PluginRegistriesBeanDefinitionRegistrar.getTargetType(PluginRegistriesBeanDefinitionRegistrar.java:101)
	at org.springframework.plugin.core.config.PluginRegistriesBeanDefinitionRegistrar.registerBeanDefinitions(PluginRegistriesBeanDefinitionRegistrar.java:71)
	at org.springframework.context.annotation.ImportBeanDefinitionRegistrar.registerBeanDefinitions(ImportBeanDefinitionRegistrar.java:86)
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.lambda$loadBeanDefinitionsFromRegistrars$1(ConfigurationClassBeanDefinitionReader.java:396)
	at java.base/java.util.LinkedHashMap.forEach(LinkedHashMap.java:684)
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsFromRegistrars(ConfigurationClassBeanDefinitionReader.java:395)
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsForConfigurationClass(ConfigurationClassBeanDefinitionReader.java:157)
	at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitions(ConfigurationClassBeanDefinitionReader.java:129)
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.processConfigBeanDefinitions(ConfigurationClassPostProcessor.java:343)
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(ConfigurationClassPostProcessor.java:247)
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanDefinitionRegistryPostProcessors(PostProcessorRegistrationDelegate.java:311)
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:112)
	at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:746)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:564)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:734)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:408)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:308)
	at org.springframework.boot.test.context.SpringBootContextLoader.loadContext(SpringBootContextLoader.java:132)
	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContextInternal(DefaultCacheAwareContextLoaderDelegate.java:141)
	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:90)
	at org.springframework.test.context.support.DefaultTestContext.getApplicationContext(DefaultTestContext.java:124)
	at org.springframework.test.context.web.ServletTestExecutionListener.setUpRequestContextIfNecessary(ServletTestExecutionListener.java:190)
	at org.springframework.test.context.web.ServletTestExecutionListener.prepareTestInstance(ServletTestExecutionListener.java:132)
	at org.springframework.test.context.TestContextManager.prepareTestInstance(TestContextManager.java:248)
	at org.springframework.test.context.junit.jupiter.SpringExtension.postProcessTestInstance(SpringExtension.java:138)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.lambda$invokeTestInstancePostProcessors$8(ClassBasedTestDescriptor.java:363)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.executeAndMaskThrowable(ClassBasedTestDescriptor.java:368)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.lambda$invokeTestInstancePostProcessors$9(ClassBasedTestDescriptor.java:363)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:195)
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:177)
	at java.base/java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1654)
	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:484)
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:474)
	at java.base/java.util.stream.StreamSpliterators$WrappingSpliterator.forEachRemaining(StreamSpliterators.java:312)
	at java.base/java.util.stream.Streams$ConcatSpliterator.forEachRemaining(Streams.java:735)
	at java.base/java.util.stream.Streams$ConcatSpliterator.forEachRemaining(Streams.java:734)
	at java.base/java.util.stream.ReferencePipeline$Head.forEach(ReferencePipeline.java:658)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.invokeTestInstancePostProcessors(ClassBasedTestDescriptor.java:362)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.lambda$instantiateAndPostProcessTestInstance$6(ClassBasedTestDescriptor.java:283)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.instantiateAndPostProcessTestInstance(ClassBasedTestDescriptor.java:282)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.lambda$testInstancesProvider$4(ClassBasedTestDescriptor.java:272)
	at java.base/java.util.Optional.orElseGet(Optional.java:369)
	at org.junit.jupiter.engine.descriptor.ClassBasedTestDescriptor.lambda$testInstancesProvider$5(ClassBasedTestDescriptor.java:271)
	at org.junit.jupiter.engine.execution.TestInstancesProvider.getTestInstances(TestInstancesProvider.java:31)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$prepare$0(TestMethodTestDescriptor.java:102)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.prepare(TestMethodTestDescriptor.java:101)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.prepare(TestMethodTestDescriptor.java:66)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$prepare$2(NodeTestTask.java:123)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.prepare(NodeTestTask.java:123)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:90)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1540)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:35)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:54)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:107)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:88)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.lambda$execute$0(EngineExecutionOrchestrator.java:54)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.withInterceptedStreams(EngineExecutionOrchestrator.java:67)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:52)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:114)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:86)
	at org.junit.platform.launcher.core.DefaultLauncherSession$DelegatingLauncher.execute(DefaultLauncherSession.java:86)
	at org.junit.platform.launcher.core.SessionPerRequestLauncher.execute(SessionPerRequestLauncher.java:53)
	at org.gradle.api.internal.tasks.testing.junitplatform.JUnitPlatformTestClassProcessor$CollectAllTestClassesExecutor.processAllTestClasses(JUnitPlatformTestClassProcessor.java:99)
	at org.gradle.api.internal.tasks.testing.junitplatform.JUnitPlatformTestClassProcessor$CollectAllTestClassesExecutor.access$000(JUnitPlatformTestClassProcessor.java:79)
	at org.gradle.api.internal.tasks.testing.junitplatform.JUnitPlatformTestClassProcessor.stop(JUnitPlatformTestClassProcessor.java:75)
	at org.gradle.api.internal.tasks.testing.SuiteTestClassProcessor.stop(SuiteTestClassProcessor.java:61)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:36)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:24)
	at org.gradle.internal.dispatch.ContextClassLoaderDispatch.dispatch(ContextClassLoaderDispatch.java:33)
	at org.gradle.internal.dispatch.ProxyDispatchAdapter$DispatchingInvocationHandler.invoke(ProxyDispatchAdapter.java:94)
	at com.sun.proxy.$Proxy2.stop(Unknown Source)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker$3.run(TestWorker.java:193)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.executeAndMaintainThreadName(TestWorker.java:129)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.execute(TestWorker.java:100)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.execute(TestWorker.java:60)
	at org.gradle.process.internal.worker.child.ActionExecutionWorker.execute(ActionExecutionWorker.java:56)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:133)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:71)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:69)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:74)
Caused by: java.lang.ClassNotFoundException: org.springframework.plugin.metadata.SimplePluginMetadata
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:582)
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521)
```





## - 원인

- `ClassNotFoundException`이 먼저 뜨고
- 이후에 `NoClassDefFoundError`
  - `NoClassDefFoundError`는 `ClassNotFoundException`과 다르게 jar파일과 관련이 있거나, 컴파일 시점에 존재했던 클래스가 런타임에 존재하지 않으면 발생하는 에러라고 함.
- ErrorLog를 확인해보면 `org/springframework/plugin/metadata/SimplePluginMetadata`
  - [Jar파일 다운로드 관련 링크](https://jar-download.com/artifacts/org.springframework.plugin/spring-plugin-metadata/1.0.0.RELEASE/source-code/org/springframework/plugin/metadata/SimplePluginMetadata.java)





## - 해결

- Dependency 추가

  ```java
  implementation 'org.springframework.plugin:spring-plugin-metadata:1.0.0.RELEASE'
  ```



------

> 마지막 수정일시: 2022-09-29 13:37