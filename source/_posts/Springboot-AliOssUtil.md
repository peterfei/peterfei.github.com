title: Springboot AliOssUtil
date: 2021-03-22 14:05:32
tags:
---

   - oss

     



```java
package com.xxx.common.utils;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.util.IdUtil;
import com.aliyun.oss.*;
import com.aliyun.oss.model.CannedAccessControlList;
import com.aliyun.oss.model.PutObjectRequest;
import com.aliyun.oss.model.PutObjectResult;
import com.huazhu.spm.face.config.AliOssProperties;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.io.InputStream;
import java.util.Date;

/**
 * @author haobiao
 * @date 2020/12/22 11:05
 * @description 阿里OSS存储
 */
@Component
@Slf4j
public class AliOssUtil {

    private AliOssProperties aliOssProperties;

    private OSS initClient() {
        ClientBuilderConfiguration conf = new ClientBuilderConfiguration();
        conf.setConnectionTimeout(10000);
        conf.setMaxErrorRetry(10);
        return new OSSClientBuilder().build(aliOssProperties.getEndpoint(), aliOssProperties.getKey(), aliOssProperties.getSecret(), conf);
    }

    /**
     * 上传照片
     *
     * @param inputStream
     * @param fileType    文件类型
     * @return
     */
    public String ossUploadFile(InputStream inputStream, String fileType) {
        try {
            OSS ossBuild = new OSSClientBuilder().build(aliOssProperties.getEndpoint(), aliOssProperties.getKey(), aliOssProperties.getSecret());
            String bucketName = aliOssProperties.getBucket();
            String endpoint = aliOssProperties.getEndpoint();
            String fileName = DateUtil.format(new Date(), "yyyyMMdd") + "/" + IdUtil.simpleUUID() + "." + fileType;
            //上传文件
            PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, fileName, inputStream);
            PutObjectResult result = ossBuild.putObject(putObjectRequest);
            //设置权限为公开读
            ossBuild.setObjectAcl(bucketName, fileName, CannedAccessControlList.PublicRead);
            if (null != result) {
                return "http://" + bucketName + "." + endpoint + "/" + fileName;
            }
            ossBuild.shutdown();
        } catch (Exception e) {
            log.error("ossUploadFile exception is {}", e);
        }
        return "";
    }


    public AliOssUtil(AliOssProperties aliOssProperties) {
        this.aliOssProperties = aliOssProperties;
    }
}
```

