---
layout: post
title: AWS-SDK S3 Presigned URL 파일 다운로드
excerpt: AWS SDK 로 Presgiend URL 생성하고 다운로드 구현
date: 2022-07-22
tags: [typescript, aws, s3, nextjs]
toc: true
---

## 1. 개요

AWS SDK 를 이용하여 파일 다운로드를 위한 Presgiend URL을 생성하고 생성한 URL을 통해 파일 다운로드 구현.

## 2. 구성 및 구현

### 2.1 S3 client

Region, Secret 정보등을 세트하여 client를 생성 하고 export 한다.

```typescript
// Create service client module using ES6 syntax.
import { S3Client } from "@aws-sdk/client-s3";
// Set the AWS Region.
const REGION = process.env.S3_REGION; //e.g. "us-east-1"
const accKey = process.env.S3_ACCESS_KEY;
const secretAccKey = process.env.S3_SECRET_ACCESS_KEY;
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

### 2.2 API (nextjs)

S3 client, presigner 를통해 Presgiend URL을 생성한다.

```typescript
import { NextApiRequest, NextApiResponse } from "next";
import { GetObjectCommand } from "@aws-sdk/client-s3";
import { HttpMethod } from "@src/datas/constants";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
// 위에 생성한 s3Client의 경로
import s3Client from "lib/s3Client";
import path from "path";

const S3GetDownloadUrl = async (req: NextApiRequest, res: NextApiResponse) => {
  try {
    if (req.method.toUpperCase() !== HttpMethod.POST) {
      throw "BAD_REQUEST";
    }

    let { fileName, filePath } = req.body;

    let pathKey = path.join(filePath, fileName);
    // windows 환경에서 path는 \를 이용하여 구분 하지만 s3에선 '/'로 구분한다.
    pathKey = (pathKey as string).replaceAll("\\", "/");

    console.log(`S3 download file info : ${pathKey}`);

    const exp = parseInt(process.env.S3_PRESIGNED_URL_EXPIRATION_SECONDS) || 70;
    const bucketParams = {
      Bucket: process.env.S3_BUCKET_NAME,
      Key: pathKey,
    };

    const getObjectCommand = new GetObjectCommand(bucketParams);
    const url = await getSignedUrl(s3Client, getObjectCommand, {
      expiresIn: exp,
    });

    console.log(`Generated url: ${url}`);
    res.json({
      url: url,
      fileName: fileName,
      filePath: filePath,
    });
  } catch (err) {
    res.json(err);
  }
};

export default S3GetDownloadUrl;
```

### 2.3 Client FileService

클라이언트단에서 API에 URL을 요청하고 다운로드를 진행하는 Service 이다. Upload, Delete등의 기능들도 같은 파일에 구현함.

```typescript
export type FileAPIOptions = {
  token?: any;
  fileName?: string;
  filePath?: string;
  repository?: FileRepository;
};

export const downloadFileS3 = async (
  options: FileAPIOptions,
  onDownloadProgress?: (e: ProgressEvent) => void
) => {
  try {
    const result = await axios
      .post(
        `${API_URL.S3_DOWNLOAD_FILE_URL}`,
        {
          fileName: options.fileName,
          filePath: options.filePath,
        },
        {
          headers: {
            // 유저 인증이 필요한경우
            authorization: options.token,
          },
        }
      )
      .then((res) => res.data)
      .catch((err) => {
        throw err;
      });

    return await axios
      .get(result.url, {
        onDownloadProgress,
        responseType: "blob",
      })
      .then((res) => {
        const url = window.URL.createObjectURL(new Blob([res.data]));
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute("download", options.fileName);
        document.body.appendChild(link);
        link.click();
      });
  } catch (err) {
    throw err;
  }
};
```

### 2.4 UI

UI 단에서 Fileservice의 downloadFileS3 를 호출하면 된다.

```tsx
const onFileDownloadClick = (e: any) => {
  downloadFileS3(
    {
      fileName: "<your_file_name>",
      filePath: "<your_file_path>",
      token: "<authorization_token>",
    },
    // progress 처리가 필요할 경우
    (e) => {
      // progress
    }
  );
};

return (
  <div>
    <button onClick={onFileDownloadClick}>Download</button>
  </div>
);
```
