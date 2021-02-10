---
title: Product Flavors
date: 2016-09-22 12:33:37
category: Android
tags: 
    - Android
---
# 使用
## 添加 productFlavors

```
productFlavors {
    production {
    }
    debug {
    }
}
```
## 指定包名
```
productFlavors {
    debug {
	    applicationId "com.xxx.xxx.debug"
    }
}
```
## 添加manifestPlaceholders
```
productFlavors {
    debug {
	    applicationId "com.xxx.xxx.debug"
	    manifestPlaceholders = [UMENG_CHANNEL_VALUE: "web"]
    }
    // 批量处理，为每个flavor添加manifestPlaceholders
    productFlavors.all { 
        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name] 
    }
}
```
##添加buildConfigFiled
```
buildConfigField "boolean", "Production", "false"
```
##每次打包吧mapping文件拷贝出来
```
    applicationVariants.all { variant ->
        if (variant.getBuildType().isMinifyEnabled()) {
            variant.assemble.doLast {
                copy {
                    from variant.mappingFile
                    into "${rootDir}/mappings"
                    rename { String fileName ->
                        "mapping-${variant.name}-${variant.versionCode}.txt"
                    }
                }
            }
        }
    }
```