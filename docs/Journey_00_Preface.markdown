### This version of this chapter was part of our working repository during the project. The final version of this chapter is now available on MSDN at [http://aka.ms/cqrs](http://aka.ms/cqrs).

> # Preface
# 前書き

> _Why are we embarking on this journey?_

_何故私たちはこの旅をするのか？_


> "The best way to observe a fish is to become a fish," Jacques Cousteau

> 魚を観察する最良の方法は魚になることである。 Jacques Cousteau

> # Why we created this guidance now

# どうしてこのガイダンスを作成したのか

> The **Command Query Responsibility Segregation (CQRS)** pattern and 
> **Event Sourcing** (ES) are currently generating a great deal of interest 
> from developers and architects who are designing and building 
> large-scale, distributed systems. There are conference sessions, blogs, 
> articles, and frameworks all dedicated to the CQRS pattern and to event 
> sourcing, and all explaining how they can help you to improve the 
> maintainability, testability, scalability, and flexibility of your 
> systems. 

**コマンド・クエリ責務分離（CQRS, Command Query Responsibility Segregation）** と
今日、イベントソーシング（ES, Event Sourcing）は大規模な分散システムを設計・構築している
開発者やアーキテクトから大きな関心を集めています。カンファレンスセッション、ブログ、記事、
フレームワークなど、すべてがCQRSパターンとイベントソーシングに着目しており、
これがシステムの保守性、テスト性、スケーラビリティ、柔軟性の向上にどのように役立つのかを説明しています。

> However, like anything new, it takes some time before a pattern, 
> approach, or methodology is fully understood and consistently defined by 
> the community and has useful, practical guidance to help you to apply or
> implement it. 

しかし、新しいものは何でもそうですが、コミュニティがパターン、アプローチ、手法を完全に
理解し、一貫した定義を行い、それを適用・実装するのに役立つ実用的なガイダンスができるまでには時間がかかります。

> This guidance is designed to help you get started with the CQRS pattern 
> and event sourcing. It is not intended to be **the** guide to the CQRS 
> pattern and event sourcing, but **a** guide that describes the 
> experiences of a development team in implementing the CQRS pattern and 
> event sourcing in a real-world application. The development team did not 
> work in isolation; they actively sought input from industry experts and 
> from a wider group of advisors to ensure that the guidance is both 
> detailed and practical. 

このガイダンスは、CQRSパターンとイベントソーシングの適用を始めるのに役立つように設計されています。
本ガイドは、CQRSパターンとイベント・ソーシングの**唯一の**ガイドではなく、
CQRSパターンとイベント・ソーシングを実際のアプリケーションに実装した開発チームの経験を記した**1つ**のガイドです。
また、詳細かつ実用的なガイドになるよう、開発チームの経験だけではなく、業界の専門家やより多くのアドバイザーに積極的に意見を求めました。

> The CQRS pattern and event sourcing are not mere simplistic solutions to 
> the problems associated with large-scale, distributed systems. By 
> providing you with both a working application and written guidance, we 
> expect you'll be well prepared to embark on your own CQRS journey. 

CQRSパターンとイベント・ソーシングは、大規模な分散システムに関連する問題を解決するための単なる単純なソリューションではありません。
本著では、実際に動作するアプリケーションとガイドの両方を活用した、読者がCQRSを巡る旅を始める準備を行うお手伝いができればと考えています。

> # How is this guidance structured?

# ガイダンスの構成

> There are two closely related parts to this guidance: 

このガイダンスには密接に関連した2つのパートがあります。

> * A working reference implementation (RI) sample, which is intended to
>   illustrate many of the concepts related to the CQRS pattern and event
>   sourcing approaches to developing complex enterprise
>   applications. 

* 動作する参照実装。このサンプルは、複雑なエンタープライズ・アプリケーションを開発する際のCQRSパターンとイベント・ソーシング・アプローチに関連する多くの概念を説明することを目的としています。

> * This written guidance, which is intended to complement the RI by 
>  describing how it works, what decisions were made during its 
>  development, and what trade-offs were considered. 

* 文章で書かれたガイダンス。（このドキュメント）参照実装がどのように機能し、開発時にどのような決定がなされたか、
どのようなトレードオフが考慮されたかを説明し、参照実装を補完することを目的としています。

> This written guidance is itself split into three distinct sections that you can read independently: a description of the journey we took as we learned about CQRS, and a collection of CQRS reference materials, and a collection of case studies that describe the experiences other teams have had with the CQRS pattern. The map in Figure 1 illustrates the relationship between the first two sections: a journey with some defined stopping points that enables us to explore a space. 

このガイダンスはそれぞれ独立して読めるように構成した3つのセクションに分かれています。1つはCQRSを学ぶ旅の道のりの説明、もう1つはCQRSの参考資料集、
もう1つはCQRSパターンを使って他のチームが経験したことを説明するケーススタディ集です。 図1の地図は、最初の2つのセクションの関係を示しています。それは、ある空間を探索することができる、いくつかの定義された停止点を持つ旅です。

![Figure 1][fig1]

> **A CQRS journey**

**CQRSを巡る旅**

> ## A CQRS journey

## CQRSを巡る旅

> This section is closely related to the RI and the chapters follow the 
> chronology of the project to develop the RI. Each chapter describes 
> relevant features of the domain model, infrastructure elements, 
> architecture, and user interface (UI) that the team was concerned with 
> during that phase of the project. Some parts of the system are discussed 
> in several chapters, and this reflects the fact that the team revisited 
> certain areas during later stages. Each of these chapters discuss how 
> and why particular CQRS patterns and concepts apply to the design and 
> development of particular bounded contexts, describe the implementation, 
> and highlight any implications for testing. 

このセクションは参照実装と密接に関連しています。各章は参照実装を開発するプロジェクトについて時系列に沿って書かれています。
各章では、プロジェクトのその段階でチームが関心を持ったドメインモデル、インフラストラクチャー要素、アーキテクチャー、
ユーザーインターフェース（UI）の関連機能について説明しています。システムの一部は複数の章で説明されていますが、
これはチームが後の段階で特定の分野を再検討したことを反映しています。それぞれの章では、特定のCQRSパターンやコンセプトを
特定の境界づけられたコンテキストの設計と開発に適用する方法と理由について議論し、実装例とテストへの影響についても言及しています。

> Other chapters look at the big picture. For example, there 
> is a chapter that explains the rationale for splitting the RI into the 
> bounded contexts we chose, another chapter analyzes the implications of 
> our approach for versioning the system, and other chapters look at how 
> the different bounded contexts in the RI communicate with each other. 

その他の章では、全体像を紹介します。例えば、ある境界づけられたコンテキストに参照実装において分割を行った根拠を説明する章や、
システムのバージョン管理に対する私たちのアプローチの意味を分析する章、参照実装の異なる境界づけられたコンテキストがどのように
相互通信するかを考える章などがあります。

> This section describes our journey as we learned about CQRS, and how we 
> applied that learning to the design and implementation of the RI. It is 
> not prescriptive guidance and is not intended to illustrate the only way 
> to apply the CQRS approach to our RI. We have tried wherever possible to 
> capture alternative viewpoints through consultation with our advisors 
> and to explain why we made particular decisions. You may disagree with 
> some of those decisions; please let us know at 
> [cqrsjourney@microsoft.com][cqrsemail]. 

このセクションでは、旅を通じて私たちが学んだことと、それを参照実装の設計と実装に適用した方法を紹介します。
ただし、これは必ずこうしなければならないという指針ではなく、参照実装にCQRSを適用する唯一の方法を示すことは意図していません。
ここで私たちが下した決定については、アドバイザーとの協議を通じて、可能な限り代替的な視点を取り入れ、その理由を説明するように努めました。
この本で紹介した決定に納得できない場合は、[cqrsjourney@microsoft.com][cqrsemail]まで連絡をいただけたらと思います。

> This section of the written guidance makes frequent cross-references to the material in the second section for readers who wish to explore any of the concepts or patterns in more detail. 

このセクションでは、コンセプトやパターンの詳細を知りたい読者のために、第2セクションの資料を頻繁に相互参照するようにしています。

> ## CQRS reference

## CQRS参考資料集

> The second section of the written guidance is a collection of reference 
> material collated from many sources. It is not the definitive 
> collection, but should contain enough material to help you to understand 
> the core patterns, concepts, and language of CQRS.

ガイダンスの第2部は、多くの情報源から集めた参考資料をまとめたものです。
これらの資料は決定的なものではありませんが、CQRSの中核となるパターン、コンセプト、語彙を理解するのに十分な資料集となっています。

> ## Tales from the trenches

## 塹壕物語

> This section of the written guidance is a collection of case studies from other teams that describe their experiences of implementing the CQRS pattern and event sourcing in the real world. These case studies are not as detailed as the journey section of the guidance and are intended to give an overview of these projects and to summarize some of the key lessons learned.

このセクションでは、CQRSパターンとイベントソーシングを実際に導入した他のチームのケーススタディを紹介します。
このセクションのケーススタディは、ガイダンスの旅のセクションほど詳細ではなく、これらのプロジェクトの概要を説明し、得られた重要な教訓を要約することを目的としています。

> The following is a list of the chapters that comprise both sections of 
> the written guidance: 

以下は2つのセクションを構成する章の一覧です。

> ### A CQRS journey
> 
> * Chapter 1, "The Contoso Conference Management System," introduces our
>   sample application and our team of (fictional) experts.
> * Chapter 2, "Decomposing the Domain," provides a high-level view of the
>   sample application and describes the bounded contexts that make up the
>   application.
> * Chapter 3, "Orders and Registrations Bounded Context," introduces our
>   first bounded context, explores some CQRS concepts, and describes some
>   elements of our infrastructure.
> * Chapter 4, "Extending and Enhancing the Orders and Registrations
>   Bounded Context," describes adding new features to the bounded context
>   and discusses our testing approach.
> * Chapter 5, "Preparing for the V1 Release," describes adding two new
>   bounded contexts and handling integration issues between them, and
>   introduces our event-sourcing implementation. This is our first
>   pseudo-production release.
> * Chapter 6, "Versioning Our System," discusses how to version the
>   system and handle upgrades with minimal down time.
> * Chapter 7, "Adding Resilience and Optimizing Performance," describes
>   what we did to make the system more resilient to failure scenarios and
>   how we optimized the performance of the system. This was the last
>   release of the system in our journey.
> * Chapter 8, "Lessons Learned," collects the key lessons we learned from
>   our journey and suggests how you might continue the journey.

### CQRSを巡る旅
* 第1章 "[コントソ](https://ja.wikipedia.org/wiki/コントソ)会議システム" - サンプルアプリケーションと架空の専門家の紹介
* 第2章, "ドメインの分解" - サンプルアプリケーションの大枠を示し、アプリケーションを構成する境界付けられたコンテキストについて説明します。
* 第3章, "注文と登録の境界付けられたコンテキスト" - 最初の境界付けられたコンテキストを通じてCQRSのコンセプトを学び、いくつかのインフラ要素についても説明します。
* 第4章, "注文と登録の境界付けられたコンテキストの拡張と機能追加" - 境界付けられたコンテキストに対する機能追加とテストの手法について議論します。
* 第5章, "バージョン1リリースに向けて" 2つの新しい境界付けられたコンテキストを追加し、これらの統合を行う方法について検討し、イベントソーシングの実装を紹介します。この実装は疑似プロダクションの最初のリリースになります。
* 第6章, "システムのバージョン管理" - システムのバージョン管理とダウンタイムが最小となるアップグレードの方法について議論します。
* 第7章, "レジリエンスの追加とパフォーマンス最適化" システムが障害シナリオに対してレジリエンス(回復性)を持つために行ったことや、システムのパフォーマンスの最適化の方法について述べます。これは、CQRSを巡る旅における最終リリースになります。
* 第8章, "教訓" 旅を通じて学んだ教訓をまとめ、読者が自分で旅を続けていくための方法を提案します。

> ### CQRS Reference

> * Chapter 1, "CQRS in Context," provides some context for CQRS,
>   especially in relation to the domain-driven design approach.
> * Chapter 2, "Introducing the Command Query Responsibility Segregation
>   Pattern," provides a conceptual overview of the CQRS pattern.
> * Chapter 3, "Introducing Event Sourcing," provides a conceptual
>   overview of event sourcing.
> * Chapter 4, "A CQRS and ES Deep Dive," describes the CQRS pattern and
>   event sourcing in more depth.
> * Chapter 5, "Communicating between Bounded Contexts," describes some
>   options for communicating between bounded contexts.
> * Chapter 6, "A Saga on Sagas," explains our choice of terminology:
>   process manager instead of saga. It also describes the role of process
>   managers.
> * Chapter 7, "Technologies Used in the Reference Implementation,"
>   provides a brief overview of some of the other technologies we used,
>   such as the Windows Azure Service Bus.
> * Appendix 1, "Building and Running the Sample Code," contains detailed
>   instructions for downloading, building, and running the sample
>   application and test suites.
> * Appendix 2, "Migrations," contains instructions for performing the code and data migrations between the pseudo-production releases of the Contoso Conference Management System.

### CQRS参考資料集

* 第1章, "CQRSのコンテキスト" - CQRSに関するいくつかのコンテキスト、特にドメイン駆動設計について説明します。
* 第2章, "コマンド・クエリ責務分離(CQRS)パターン" - CQRSパターンの概念的な概要を紹介します。
* 第3章, "イベントソーシング" - イベントソーシングの概念的な概要を紹介します。
* 第4章, "CQRSとESの深堀り" - CQRSとイベントソーシングについてもう少し詳しく見ていきます。
* 第5章, "境界付けられたコンテキスト同士の通信" - 境界付けられたコンテキスト同士の通信について、いくつかの方法を検討します。
* 第6章, "サーガ・パターンにおけるサーガ" 用語について、"サーガ"ではなく"プロセスマネージャー"という用語を選んだことを説明します。 また、プロセスマネージャーの役割についても説明します。
* 第7章, "参照実装に使用した技術" - Windows Azure Service Busといった参照実装に使用した他の技術の概要を紹介します。
* 付録 1, "サンプルコードのビルドと実行" - サンプルアプリケーションとテストコードのダウンロード、ビルド、実行に関する詳細です。
* 付録 2, "移行" コンソト会議システムの疑似プロダクションリリース間のコードとデータの移行を行うための手順について書かれています。

> ### Tales from the trenches
> 
> * Chapter 1, "Twilio," describes a highly available, cloud-hosted, communications platform. Although the team who developed this product did not explicitly use CQRS, many of the architectural concepts they adopted are very closely related to the CQRS pattern.
> * Chapter 2, "Lokad Hub," describes a project that made full use of domain-driven design, CQRS, and event sourcing in an application designed to run on multiple cloud platforms.
> * Chapter 3, "DDD/CQRS for large financial company," describes a project that made full use of domain-driven design and CQRS to build a reference application for a large financial company. It used CQRS to specifically address the issues of performance, scalability, and reliability.
> * Chapter 4, "Digital Marketing," describes how an existing application was refactored over time while delivering new features. This project adopted the CQRS pattern for one of its pieces as the project progressed.
> * Chapter 5, "TOPAZ Technologies," describes a project that used the CQRS pattern and event sourcing to simplify the development of an off-the-shelf enterprise application.
> * Chapter 6, "eMoney Nexus," describes a project to migrate an application that used legacy three-tier architecture to an architecture that used the CQRS pattern and event sourcing. Many of the conclusions drawn in this project are similar to our own experiences in our CQRS journey.

### 塹壕物語

* 第1章, "Twilio," - Twilioはクラウドにホスティングされた高可用性のあるコミュニケーションプラットフォームです。製品の開発チームはCQRSを明示的には使用していませんが、チームが採用したアーキテクチャコンセプトの多くはCQRSパターンと非常に密接な関連があります。 
* 第2章, "Lokad Hub" - Lokad Hubにおいて、複数のクラウドプラットフォームで動作するアプリケーションを設計するため、ドメイン駆動設計やCQRS、イベントソーシングを駆使したプロジェクトの事例を紹介します。
* 第3章, "大手金融会社のDDD/CQRS" - ドメイン駆動設計とCQRSを駆使して、大手金融会社のリファレンス・アプリケーションを構築したプロジェクトの事例を紹介します。このプロジェクトでは特に、パフォーマンス、スケーラビリティ、および信頼性の問題についてCQRSが解決に役立ちました。
* 第4章, "デジタルマーケティング" - 既存のアプリケーションを新機能を提供しながら時間をかけてリファクタリングしていく事例を紹介します。このプロジェクトでは、プロジェクトの進行に合わせて、部品の1つとしてCQRSパターンを採用しました。
* 第5章, "TOPAZテクノロジー" - CQRSパターンとイベントソーシングを利用して、既製のエンタープライズアプリケーションの開発を簡素化したプロジェクトについて説明します。
* 第6章, "eMoney Nexus" - レガシーな3層アーキテクチャを使用したアプリケーションを、CQRSパターンとイベントソーシングを使用したアーキテクチャに移行するプロジェクトについて説明します。このプロジェクトから導出された結論の多くは、私たちがCQRSを導入した際の経験と似ています。

> ## Selecting the domain for the RI

## 参照実装のためのドメインの選択

> Before embarking on our journey, we needed to have an outline of the 
> route we planned to take and an idea of what the final destination 
> should be. We needed to select an appropriate domain for the RI. 

旅の前に、最終目的地と経路について、概要を把握しておく必要があります。
そのために、参照実装では適切なドメインを選択しなければいけません。

> We engaged with the community and our advisory board to help us choose a 
> domain that would enable us to highlight as many of the features and 
> concepts of CQRS as possible. To help us select between our candidate 
> domains, we used the criteria in the following list. The domain selected 
> should be: 

執筆にあたって、CQRSの特徴やコンセプトをできるだけ多く取り上げることができるようなドメインを選ぶために、
コミュニティや諮問委員会の協力を得ました。
その結果、以下の基準を用いてドメインを選択することにしました。
参照実装のドメインは以下のような特徴を持ちます。

> * **Non-trivial.** The domain must be complex enough to exhibit real 
> problems, but at the same time simple enough for most people to 
> understand without weeks of study. The problems should involve dealing 
> with temporal data, stale data, receiving out-of-order events, and 
> versioning. The domain should enable us to illustrate solutions using 
> event sourcing, sagas, and event merging. 

* **自明ではないこと** ドメインは、実際の問題を示すのに十分な程度複雑であると同時に、
ほとんどの人が何週間も勉強しなくても理解できる程度に単純である必要があります。
ドメインは、時系列データ、データの陳腐化、順序に反したイベントの受信、バージョニングへの対応といった問題を含むようにします。
ドメインを通じて、イベントソーシング、サーガ、イベントマージを使ったソリューションを説明できなければいけません。

> * **Collaborative.** The domain must contain collaborative elements where 
> multiple actors can operate simultaneously on shared data. 

* **コラボレーション** ドメインは複数のアクターが共有データを同時に操作するようなコラボレーション要素を持ちます。

> * **End to end.** We wanted to be able illustrate the concepts and 
> patterns in action from the back-end data store through to the user 
> interface. This might include disconnected mobile and smart 
> clients. 

* **エンド・ツー・エンド** バックエンドのデータストアからユーザーインターフェースに至るまで、
コンセプトやパターンの動作を説明できるようにします。これには、切断されたモバイルやスマートクライアントも含みます。


> * **Cloud friendly.** We wanted to have the option of hosting parts of the 
> RI on Windows Azure and be able to illustrate how you can use CQRS for 
> cloud-hosted applications. 

* **クラウド・フレンドリー** Windows Azure上で参照実装の一部をホスティングするオプションを持ち、クラウド・ホスティングされたアプリケーションにCQRSを使用する方法を説明できるようにしました。

> * **Large.** We wanted to be able to show how our domain can be broken 
> down into multiple bounded contexts to highlight when to use and when 
> not use CQRS. We also wanted to illustrate how multiple architectural 
> approaches (CQRS, CQRS/ES, and CRUD) and legacy systems can co-exist 
> within the same domain. We also wanted to show how multiple 
> development teams could carry out work in parallel. 

* **十分な規模** 私たちは、CQRSを採用する場合そうではない場合を比較するため、
ドメインを複数の境界付けられたコンテキストに分解できる例となるようにしました。
また、複数のアーキテクチャアプローチ（CQRS、CQRS/ES、CRUD）やレガシーシステムが、
同じドメイン内で共存する方法を示せるようにしました。また、複数の開発チームが並行して作業を進める方法も紹介します。

> * **Easily deployable.** The RI needed to be easily deployable so that you 
> can install it and experiment with it as you read this guidance. 

* **容易なデプロイ** ガイドを読みながら参照実装をインストールして試せるように、参照実装は簡単にデプロイできなければいけません。

> As a result, we chose to implement the conference management system that 
> Chapter 1, "[The Contoso Conference Management System][j_chapter1]" introduces. 

最終的に、本著では第1章の[コントソ会議システム][j_chapter1]で紹介するカンファレンスの管理システムを採用しました。

[fig1]:           images/Map.png?raw=true
[cqrsemail]:      mailto:cqrsjourney@microsoft.com
[j_chapter1]:     Journey_01_Introduction.markdown