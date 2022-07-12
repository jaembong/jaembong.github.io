---
layout: post
title: AWS-SDK 로 Presigned URL 을 통해 S3에 파일 업로드 (Typescript)
excerpt: AWS-SDK와 Presigned URL을 통해 s3 에 파일 업로드 해보기
date: 2022-07-12
tags: [typescript, aws, s3, nextjs]
toc: true
---

## 1. S3 버킷 생성

생략

## 2. 사용자 추가

대상 버킷에 액세스할수있는 권한을 가진 사용자 및 그룹을 생성하여 Access key 와 Secret acess key 를 얻는다.

## 3. 구현

### 3.1 패키지 설치

aws-sdk 3 버전 기준으로 aws-sdk/client-s3, aws-sdk/s3-request-presigner 설치

```bash
yarn add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

### 3.2 S3 클라이언트

버킷이 생성되어 있는 Region과 access key, secret access key 를 파라미터로 S3client 생성한다.

```typescript
// Create service client module using ES6 syntax.
import { S3Client } from "@aws-sdk/client-s3";
// Set the AWS Region.
const REGION = "<your_region>"; //e.g. "us-east-1"
const accKey = "<access_key>";
const secretAccKey = "<secret_access_key>";
// Create an Amazon S3 service client object.
const s3Client = new S3Client({
  region: REGION,
  credentials: {
    accessKeyId: accKey,
    secretAccessKey: secretAccKey,
  },
});
export default s3Client;
```

### 3.3 Presigned URL 생성

Presigned URL을 생성해주는 API 코드 (nextjs)

```typescript
import { NextApiRequest, NextApiResponse } from "next";
import { PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

// 3.2에서 작성한 S3 client
import s3Client from "lib/s3Client";

const S3GetUploadUrl = async (req: NextApiRequest, res: NextApiResponse) => {
  try {
    // req에서 파일이름 가져오기
    let { fileName } = req.body;

    // url 만료 시간
    const exp = parseInt(process.env.S3_PRESIGNED_URL_EXPIRATION_SECONDS) || 70;

    /**
     * Bucket: 생성한 버킷 이름
     * Key: 버킷에 저장될 파일 이름 및 경로 (ex: dir1/dir2/filename)
     */
    const bucketParams = {
      Bucket: process.env.S3_BUCKET_NAME,
      Key: `test/${Math.ceil(Math.random() * 10 ** 10)}/${fileName}`,
    };

    // PutObjectCommand는 파일을 버킷에 Put 하기 위한 객체
    const putObjectCommand = new PutObjectCommand(bucketParams);
    const url = await getSignedUrl(s3Client, putObjectCommand, {
      expiresIn: exp,
    });

    console.log(`Generated url: ${url}`);

    // 요청한 클라이언트에 생성된 URL 전송
    res.json({ url });
  } catch (err) {
    console.log(err);
  }
};

export default S3GetUploadUrl;
```

### 3.4 URL 요청 및 업로드

```typescript
const uploadFile = async (
  file: File,
  onUploadProgress?: (e: ProgressEvent) => void
) => {
  try {
    // Presigned URL 요청
    const url = await axios
      .post(
        `<Presigned_URL_API_URL>`,
        {
          fileName: file.name,
        },
        {
          headers: {
            /* Authorization */
          },
        }
      )
      .then((res) => {
        return res.data.url;
      });

    // Presigned URL로 파일 업로드
    await axios.put(url, file, {
      headers: {
        "Content-Type": file.type,
        "Access-Control-Allow-Origin": "*",
      },
      onUploadProgress,
    });
  } catch (err) {
    console.log(err);
  }
};
```
