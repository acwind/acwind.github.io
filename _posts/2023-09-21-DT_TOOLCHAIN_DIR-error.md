---

title: 解决Xcode 15升级后的CocoaPods问题：'DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead'
categories:
- 编程人生
key: f49c35ea-1ac7-11ed-861d-0242ac120094


---


随着iOS 17的发布，Xcode也更新到了15版本。然而，在升级后，我发现一些旧项目出现了报错，问题出在CocoaPods上：

```bash
DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead
```

我在CocoaPods的官方GitHub中找到了用户提交的相关问题以及解决方案：[Issue #12012](https://github.com/CocoaPods/CocoaPods/issues/12012)

通过讨论来看，这个错误是应该是由于使用了不正确的环境变量导致的。具体来说，错误的部分在于使用了 'DT_TOOLCHAIN_DIR' 来评估 'LIBRARY_SEARCH_PATHS'，而正确的做法是使用 'TOOLCHAIN_DIR' 来替代 'DT_TOOLCHAIN_DIR'。

那么，在没有更新CocoaPods的情况下，我们该如何解决这个问题呢？其实，只需要在Podfile中加入相应的判断代码即可。比如，原始的Podfile文件内容如下：

```ruby
post_install do |installer|
    installer.generated_projects.each do |project|
          project.targets.each do |target|
              target.build_configurations.each do |config|
                    config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
               end
          end
   end
end
```

在`|config|`下面插入相应的处理代码，整个代码段变成如下：

```ruby
post_install do |installer|
    installer.generated_projects.each do |project|
          project.targets.each do |target|
              target.build_configurations.each do |config|
                  config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
                  xcconfig_path = config.base_configuration_reference.real_path
                  xcconfig = File.read(xcconfig_path)
                  xcconfig_mod = xcconfig.gsub(/DT_TOOLCHAIN_DIR/, "TOOLCHAIN_DIR")
                  File.open(xcconfig_path, "w") { |file| file << xcconfig_mod }
               end
          end
   end
end
```

最后重新运行一下`pod install`就可以解决这个问题了。希望以上的解决方案可以帮助到遇到同样问题的开发者！