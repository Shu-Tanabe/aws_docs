---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# AWS CDK

NestedStack による Stack 分割実践

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Start <carbon:arrow-right class="inline"/>
  </span>
</div>

<!-- <div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div> -->

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# AWS Cloud Develop Kit (AWS CDK)

プログラミング言語を使ってクラウドリソースを構築することができる開発キット  
ソースは OSS として公開されている[^1]

[^1]: https://github.com/aws/aws-cdk

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}

.footnotes-sep {
  @apply mt-20 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}

</style>

---

# 使用できる言語

- TypeScript
- JavaScript
- Python
- Java
- C#

---

# どのように記述するか

スタックというまとまりの中にリソースの定義を記載する

```ts {all|7|11-15|all}
/* 
  「my-bucket」というS3バケットを作成する例 
*/
import * as cdk from "@aws-cdk/core";
import * as s3 from "@aws-cdk/aws-s3";

export class AwsCdkStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const bucket = new s3.Bucket(this, "MyBucket", {
      bucketName: "my-bucket",
      versioned: true,
      websiteRedirect: { hostName: "aws.amazon.com" },
    });
  }
}
```

---

# AWS CDK の挙動

CDK はデプロイされるとまず CloudFormation Template に変換される  
その後 CloudFormation が AWS 上にリソースを作成する

<!-- ![test](/cdk-flow_100.png) -->

<img
  class="absolute top-40 left-42 right-10 bottom-0"
  src="/cdk-flow_100.png"
/>

<style>
img {
  height: 350px
}
</style>

---

# AWS CDK の挙動

先程の S3 バケットを作成する例では以下のように CloudFormation テンプレートに変換される

<div grid="~ cols-2 gap-2" m="-t-2">

### typescript

### CloudFromation Template

```ts
import * as cdk from "@aws-cdk/core";
import * as s3 from "@aws-cdk/aws-s3";

export class AwsCdkStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const bucket = new s3.Bucket(this, "MyBucket", {
      bucketName: "my-bucket",
      versioned: true,
      websiteRedirect: { hostName: "aws.amazon.com" },
    });
  }
}
```

```json
{
  "Resources": {
    "MyBucketF68F3FF0": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "my-bucket",
        "VersioningConfiguration": {
          "Status": "Enabled"
        },
        "WebsiteConfiguration": {
          "RedirectAllRequestsTo": {
            "HostName": "aws.amazon.com"
          }
        }
      },
      ・・・
```

</div>

---

# GUI の場合

<div grid="~ cols-2 gap-2" m="-t-2">

<img
  src="/GUI-left.png"
/>

<img
  src="/GUI-right.png"
/>

</div>

<style>
img {
  width: 360px
}
</style>

---

# 難しくなってない？

<style>
h1 {
  vertical-align: center;
  text-align: center;
}
</style>

---

# 何がいいのか？

- **検証環境や試験環境の構築が容易**

コマンド一つで環境構築・削除が可能

```
$ cdk deploy   // リソースを作成
$ cdk destroy  // リソースを削除
```

<br>
<br>

- **リソースの定義にプログラミング言語特有の表現を利用できる**

→ CloudFormation Template と比較してみる

---

## 課題

- プログラミング言語を習得していないと少ししんどい
- ドキュメントがまだ少ない
- 実装に関するルール決めないと複雑な構造になりやすい <scan v-click="1"> ← **今回なんとかする部分** </scan>

---

# なぜ複雑な構造になるのか？

---

# なぜ複雑な構造になるのか？

とある実装例

```ts
export class ExampleCdkStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const vpc = new ec2.Vpc(・・・);

    const subnet = new ec2.Subnet(・・・);

    const securityGroup = new ec2.SecurityGroup(・・・);

    const autoScalingGroup = new autoScaling.autoScalingGroup(・・・);

    ・・・

  }
}
```

---

# なぜ複雑な構造になるのか？

<!-- ### ・複雑な構造になる原因 ① -->

### ・1 つの Stack の中にたくさんのリソースを定義してしまっている

---

# なぜ複雑な構造になるのか？

とあるディレクトリ構成例

```
example_app
 ┣ bin
 ┃ ┗ example_app.ts 　　　　　　　　　　　　//ExampleAppStackを呼び出す
 ┣ cdk.out 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　//CloudFormation Templateが出力される
 ┣ lib
 ┃ ┣ resources
 ┃ ┃ ┣ vpc.ts
 ┃ ┃ ┣ igw.ts
 ┃ ┃ ┣ subnet.ts
 ┃ ┃ ┣ securityGroup.ts
 ┃ ┃ ・・・
 ┃ ┗ example_app-stack.ts //ExampleAppStack内にリソースを定義する
 ┗ cdk.json 　　　　　　　　　　　　　　　　　　　　　　　　　　　　//設定値の定義など
```

---

# なぜ複雑な構造になるのか？

<div class="gray-out">

### ・1 つの Stack の中にたくさんのリソースを定義してしまっている

</div>
<br>
<br>
<br>

### ・編集したいリソースが探しづらい構造になっている

<style>
.gray-out {
  color: #BDBDBD;
}

</style>

---

# なぜ複雑な構造になるのか？

リソースを定義したとあるクラス間の関係

<img
  class="absolute top-30 left-80 right-10 bottom-0"
  src="/frontend-stack-bad-architecture_class.png"
/>

<style>
img {
  height: 400px;
}
</style>

---

# なぜ複雑な構造になるのか？

<div class="gray-out">

### ・1 つの Stack の中にたくさんのリソースを定義してしまっている

<br>
<br>
<br>

### ・編集したいリソースが探しづらい構造になっている

</div>

<br>
<br>
<br>

### ・1 つの Stack 内で複雑なクラス間の依存関係が形成されている

### &nbsp; &nbsp; → 複数名で開発する場合に責任範囲の切り分けが難しい

<style>
.gray-out {
  color: #BDBDBD;
}

</style>

---

# 複雑な構造になる原因まとめ

### ・1 つの Stack の中にたくさんのリソースを定義してしまっている

<br>
<br>
<br>

### ・編集したいリソースが探しづらい構造になっている

<br>
<br>
<br>

### ・1 つの Stack 内で複雑なクラス間の依存関係が形成されている

### &nbsp; &nbsp; → 複数名で開発する場合に責任範囲の切り分けが難しい

---

# 理想的な構造

### ・リソースの塊ごとに Stack が分割されている

<br>
<br>
<br>

### ・各リソースの設定がどこに記載されているかわかりやすい構造になっている

<br>
<br>
<br>

### ・各自が責任範囲を考えやすい構造になっている

---

# NestedStack を採用

以下のように Stack から別の Stack を呼ぶような Stack のネスト構造を形成できる

<img
  class="absolute top-40 left-80 right-10 bottom-0"
  src="/frontend-stack-NestedStack.png"
/>

<div v-click="1">⇨ リソースの塊ごとにStackを分割することが可能 </div>

---

# 実装方法をいったん思い出す

スタックというまとまりの中にリソースの定義を記載する

```ts
/* 
  「my-bucket」というS3バケットを作成する例 
*/
import * as cdk from "@aws-cdk/core";
import * as s3 from "@aws-cdk/aws-s3";

export class AwsCdkStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const bucket = new s3.Bucket(this, "MyBucket", {
      bucketName: "my-bucket",
      versioned: true,
      websiteRedirect: { hostName: "aws.amazon.com" },
    });
  }
}
```

---

# Stack 分割案

## SkeletonStack

VPC・Subnet などの内部ネットワーク関連リソースを定義

## FrontendStack

Route53・CloudFront・ELB など、インターネットに面したリソースを定義

## ApplicationStack

EC2、AutoScalingGroup などアプリケーションが稼働するサーバ関連のリソースを定義

## DatabaseStack

Database クラスターなどデータベースに関連するリソースを定義

---

## SecurityStack

IAM、CloudTrail、Config など、セキュリティやアクセス制御に関連するリソースを定義

## LoggingStack

FlowLog、CloudWatch Logs などログ関連のリソースを定義

---

# Stack 間の関係

Stack 同士は次のような繋がりをもつ

<img
  class="absolute top-30 left-80 right-10 bottom-0"
  src="/nested-stack-relation.png"
/>

<style>

img {
  height: 400px;
}

</style>

---

# Stack 間の関係はどのように実装しているか

実装では Stack 間の関係は以下のように表現される

<img
  class="absolute top-50 left-45 right-10 bottom-0"
  src="/frontend-stack-class-connection.png"
/>

<style>

img {
  width: 600px;
}

</style>

---

# Stack 間の関係はどのように実装しているか

実装に落とし込むと以下のようになる

```ts
export class ExampleAppStack extends cdk.Stack {
  public skeletonStack: SkeletonStack;
  public applicationStack: ApplicationStack;
  // ・・・
  const {
    vpc,
    appSubnet1a,
    appSubnet1c,
    albSecurityGroup,
    appSecurityGroup,
    ・・・
  } = this.skeletonStack;

  this.applicationStack = new ApplicationStack(scope, id, {
    vpc: vpc,
    subnets: [
      appSubnet1a,
      appSubnet1c
    ],
    appSecurityGroup,
    ・・・
  });
```

---

# 理想的な構造（ふりかえり）

### ・リソースの塊ごとに Stack が分割されている ← Clear!

<br>
<br>
<br>

### ・各リソースの設定がどこに記載されているかわかりやすい構造になっている

### &nbsp; &nbsp; ← 未対応

<br>
<br>
<br>

### ・各自が責任範囲を考えやすい構造になっている ← Clear?

---

# ディレクトリ構成

```
example_app
 ┣ bin
 ┃ ┗ example_app.ts 　　　　　　　　　　　　//ExampleAppStackを呼び出す
 ┣ cdk.out 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　//CloudFormation Templateが出力される
 ┣ lib
 ┃ ┣ skeleton
 ┃ ┃ ┣ resources
 ┃ ┃ ┃ ┣ vpc
 ┃ ┃ ┃ ┣ igw
 ┃ ┃ ┃ ┣ webSubnet
 ┃ ┃ ┃ ┣ appSubnet
 ┃ ┃ ┃ ┣ dbSubnet
 ┃ ┃ ┃ ・・・
 ┃ ┃ ┗ skeleton-stack.ts
 ┃ ┣ frontend
 ┃ ┣ application
 ┃ ┣ database
 ┃ ┣ security
 ┃ ┗ example_app-stack.ts //ExampleAppStack内にリソースを定義する
 ┗ cdk.json 　　　　　　　　　　　　　　　　　　　　　　　　　　　　//設定値の定義など
```

---

# 実際のディレクトリ構成を見てみる

<div v-click>
同じリソースでも変更理由は異なる
⇨ 種類ごとにファイルを分けた

- webSubnet
- appSubnet
- dbSubnet
- applicationLoadbalancer
- applicationListener
- applicationTarget
- and so on. . .

</div>

---

# 理想的な構造（ふりかえり）

### ・リソースの塊ごとに Stack が分割されている ← Clear!

<br>
<br>
<br>

### ・各リソースの設定がどこに記載されているかわかりやすい構造になっている

### &nbsp; &nbsp; ← Clear!

<br>
<br>
<br>

### ・各自が責任範囲を考えやすい構造になっている ← Clear!

---

# ところで

bin 直下の Stack は分割しなかった

```
example_app
 ┣ bin
 ┃ ┗ example_app.ts 　　　　　　　　　　　　//ExampleAppStackを呼び出す
 ┣ cdk.out 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　//CloudFormation Templateが出力される
 ┣ lib
 ┃ ┣ skeleton
 ┃ ┣ frontend
 ┃ ┣ application
 ┃ ┣ database
 ┃ ┣ security
 ┃ ┗ example_app-stack.ts //ExampleAppStack内にリソースを定義する
 ┗ cdk.json 　　　　　　　　　　　　　　　　　　　　　　　　　　　　//設定値の定義など
```

---

今回は root となる bin 直下の Stack の分割は、環境ごとに分けるくらい
<img
  class="absolute top-40 left-75 right-10 bottom-0"
  src="/frontend-stack-nested-stack-stages.png"
/>

<style>

img {
  height: 350px;
}

</style>

---

この場合、引数でデプロイする Stack を指定する

```
$ cdk deploy TestExampleAppStack
```

---

# おわり

---

## 参考

- [cdk コマンドの機能を 実際に叩いて理解する 【 AWS CDK Command Line Interface 】](https://dev.classmethod.jp/articles/aws-cdk-command-line-interface/)
- [JAWS DAYS2021 登壇レポート 1 年間運用して分かった CDK アンチパターン](https://chariosan.com/2021/03/20/jaws-days2021_report_anti-pattern/)

## AWS 公式

- [AWS Cloud Development Kit (CDK)](https://docs.aws.amazon.com/cdk/latest/guide/home.html)
- [AWS CDK でクラウドアプリケーションを開発するためのベストプラクティス](https://aws.amazon.com/jp/blogs/news/best-practices-for-developing-cloud-applications-with-aws-cdk/)

---

## スライド

- [Slidev](https://sli.dev/)
