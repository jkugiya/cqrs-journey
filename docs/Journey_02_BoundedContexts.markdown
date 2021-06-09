### This version of this chapter was part of our working repository during the project. The final version of this chapter is now available on MSDN at [http://aka.ms/cqrs](http://aka.ms/cqrs).

> # Chapter 2: Decomposing the Domain 

# 第2章: ドメインの分解

> _Planning the stops._

_停止計画_

>"Without stones there is no arch," Marco Polo

> "石がなければアーチを作ることはできません" Marco Polo

> In this chapter, we provide a high-level overview of the Contoso 
> Conference Management System. The discussion will help you understand 
> the structure of the application, the integration points, and how the 
> parts of the application relate to each other. 

この章では、コンソト会議管理システムの概要を説明します。
ここでの議論を通じて、アプリケーションの構造、統合ポイント、およびアプリケーションの各部分がどのように関連しているかを
理解していきましょう。

> Here we describe this high-level structure in terms borrowed from the 
> domain-driven design (DDD) approach that Eric Evans describes in his 
> book, _Domain-Driven Design: Tackling Complexity in the Heart of 
> Software_ (Addison-Wesley Professional, 2003). Although there is no 
> universal consensus that DDD is a prerequisite for implementing the 
> Command Query Responsibility Segregation (CQRS) pattern successfully, 
> our team decided to use many of the concepts from the DDD approach, such 
> as _domain_, _bounded context_, and _aggregate_, in line with common 
> practice within the CQRS community. The chapter, "[CQRS in 
> Context][r_chapter1]," in the Reference Guide discusses the relationship 
> between the DDD approach and the CQRS pattern in more detail. 

ここでは、このハイレベルな構造を、Eric Evans氏が著書「_ドメイン駆動設計: ソフトウェアの核心にある複雑さに立ち向かう_」（Addison-Wley Professional 2003）で説明しているドメイン駆動設計（DDD）の用語を使って説明します。
CQRS（Command Query Responsibility Segregation）パターンの実装を成功させるためにDDDが必須条件であるということは
普遍的なコンセンサスではありませんが、私たちのチームは、CQRSコミュニティの一般的なプラクティスに則って、_ドメイン_、
 _境界付けられたコンテキスト_、 _集約_ といったDDDの概念を沢山使用することにしました。
リファレンスガイドの「[CQRSの現状][r_chapter1]」の章では、DDDとCQRSの関係について、より詳しく説明します。

> # Definitions used in this chapter 

# この章で使用する用語

> Throughout this chapter we use a number of terms, which we'll define in 
> a moment. For more detail, and possible alternative definitions, see the 
> chapter, "[CQRS in Context][r_chapter1]," in the Reference Guide. 

本章では、いくつかの用語を使用しますが、これらの用語の定義については後ほど説明します。
詳細や別の定義については、リファレンスガイドの「[CQRSの現状][r_chapter1]」の章を参照してください。

> **Domain:** The _domain_ refers to the business domain for the Contoso 
> Conference Management System (the reference implementation). Chapter 1, 
> "[The Contoso Conference Management System][j_chapter1]," provides an 
> overview of this domain. 

**ドメイン:** _ドメイン_ は、コンソト会議管理システム（参照実装）のビジネスドメインです。第1章の「[コンソト会議管理システム][j_chapter1]」でこのドメインの概要を説明しています。

> **Bounded context:** The term _bounded context_ comes from Eric Evans' 
> book. In brief, Evans introduces this concept as a way to decompose a 
> large, complex system into more manageable pieces; a large system is 
> composed of multiple bounded contexts. Each bounded context is the 
> context for its own self-contained domain model, and has its own 
> ubiquitous language. You can also view a bounded context as an 
> autonomous business component defining clear consistency boundaries: one 
> bounded context typically communicates with another bounded context by 
> raising events. 

**境界付けられたコンテキスト:** 「境界付けられたコンテキスト」という言葉は、Eric Evansの著書に由来します。
端的に言うと、Evansはこの概念を、大規模で複雑なシステムをより管理しやすい部分に分解する方法として紹介しています。
大規模なシステムは複数の境界付けられたコンテキストで構成され、各境界付けられたコンテキストは自己完結したドメインモデルのコンテキストであり、
独自のユビキタス言語を持ちます。また、境界付けられたコンテキストを、明確な一貫性の境界を定義する自律的なビジネスコンポーネントとして捉えることもできます。つまり、境界付けられたコンテキスト同士の通信は、通常、一方のコンテキストが発生させたイベントを通じて行います。

> **GaryPersona:** When you use the CQRS pattern, you often use
> events to communicate between bounded contexts. There are alternative
> approaches to integration, such as sharing data at the database level.

> **Garyのペルソナ:** CQRSパターンでは、境界付けられたコンテキスト間の通信にイベントをよく使用します。
> 統合には、データベースレベルでデータを共有するような別のアプローチもあります。

> **Context map:** According to Eric Evans, you should "Describe the 
> points of contact between the models, outlining explicit translation for 
> any communication and highlighting any sharing." This exercise results 
> in what is called a _context map_, which serves several purposes that 
> include providing an overview of the whole system and helping people to 
> understand the details of how different bounded contexts interact with 
> each other. 

**コンテキストマップ:** Eric Evansによれば、"モデル同士の接点を記述して、 あらゆるコミュニケーションで必要となる明示的な変換について概略を述べ、
共有するものがあれば強調"しなければいけません。これを行うと、_コンテキストマップ_ と呼ばれるものができあがります。
コンテキストマップの作成には、システム全体の概要を示したり、異なる境界付けられたコンテキストが
相互作用する方法の詳細の理解に役立てたりといった、目的があります。

> # Bounded contexts in the conference management system 

# 会議管理システムにおける境界付けられたコンテキスト

**The orders and registrations bounded context:** Within the _orders and 
registrations bounded context_ are the reservations, payment, and 
registration items. When a registrant interacts with the system, the 
system creates an order to manage the reservations, payment, and 
registrations. An order contains one or more order items. 

**注文と登録のコンテキスト:** _注文と登録のコンテキスト_ には、予約、支払い、登録といった概念があります。
登録者がシステムと対話するとき、システムは予約、支払い、および登録を管理するために注文を作成します。
1つの注文には1つ以上の注文項目があります。

> A _reservation_ is a temporary reservation of one or more seats at a 
> conference. When a registrant begins the ordering process to purchase a 
> number of seats at a conference, the system creates reservations for 
> that number of seats. Those seats are then unavailable for other 
> registrants to reserve. The reservations are held for 15 minutes, during 
> which time the registrant can complete the ordering process by making a 
> payment for the seats. If the registrant does not pay for the tickets 
> within 15 minutes, the system deletes the reservation and the seats 
> become available for other registrants to reserve. 

_予約_とは、会議の1かそれ以上の座席を一時的に予約することです。登録者が会議の座席の購入のために注文プロセスを開始すると、
システムは登録者が指定した数の座席の予約を作成します。予約された座席は、他の登録者が予約することはできません。
予約は15分間保持され、その間に登録者が座席の代金を支払うと、注文プロセスが完了します。
登録者が15分以内にチケットの支払いを行わなかった場合、システムはその予約を削除し、
他の登録者が予約できるようになります。

> **CarlosPersona:** We discussed making the period of time that the
> system holds reservations a parameter that a Business Customer can
> adjust for each conference. This may be a feature that we add if we
> determine that there is a requirement for this level of control.

> **Carlosのペルソナ:** システムが予約を保持する期間を、企業のお客様が会議ごとに調整できた方がよいという話もありました。
> このレベルのコントロールが必要と判断された場合、この機能を追加するかもしれません。

> **The conference management bounded context:** Within this bounded 
> context, a business customer can create new conferences and manage them. 
> After a business customer creates a new conference, he can access the 
> details of the conference by using his email address and conference 
> locator access code. The system generates the access code when the 
> business customer creates the conference. 

**会議管理のコンテキスト:** この境界付けられたコンテキストでは、
顧客は新たに会議を作成し・管理します。 顧客は新しく会議を作成した後、
電子メールアドレスと会議のアクセスコードを使って、会議詳細にアクセスできます。
アクセスコードは、顧客が会議を作成した時にシステムが生成します。

> The business customer can specify the following information about a 
> conference: 

> * The name, description, and slug (part of the URL used to access the conference).
> * The start and end dates of the conference.
> * The different types and quotas of seats available at the conference.

顧客は会議に関する以下の情報を指定できます。

* 会議の名前、説明、スラッグ（カンファレンスに
* 会議の開始・終了時間
* 会議で利用可能な座席の種類とその数

> Additionally, the Business Customer can control the visibility of the 
> conference on the public website by either publishing or un-publishing 
> the conference. 

顧客は、会議の公開・非公開を選択して、公開されたウェブサイト上での会議の可視性をコントロールできます。

> The Business Customer can also use the conference management website to 
> view a list of orders and Attendees. 

また、顧客は、会議運営サイトから注文や参加者のリストを見ることができます。

**The payments bounded context:** The _payments bounded context_ is responsible for managing the interactions between the conference management system and external payment systems. It forwards the necessary payment information to the external system and receives an acknowledgement that the payment was either accepted or rejected. It reports the success or failure of the payment back to the conference management system. 

**支払いのコンテキスト:** _支払いのコンテキスト_ は、会議管理システムと外部の決済システムとの間の相互作用を管理する責務を負います。
このコンテキストでは、必要な支払い情報を外部システムに転送し、支払いが承認されたか拒否されたかの通知を受信し、
支払いの成功または失敗を会議管理システムに報告します。

> Initially, the Payments bounded context will assume that the Business 
> Customer has an account with the third-party payment system (although 
> not necessarily a merchant account), or that the Business Customer will 
> accept payment by invoice.

当初、支払いのコンテキストでは、顧客がサードパーティのペイメントシステムにアカウントを持っていること
（必ずしも商用アカウントである必要はない）、もしくは顧客が請求書による支払いを受諾することを想定しています。


> ## Bounded contexts not included

## その他の境界付けられたコンテキスト

> Although they didn't make it into the final release of the Contoso 
> Conference Management System, some work was done on three additional 
> bounded contexts. Members of the community are working on these and 
> other features, and any out-of-band releases and updates will be 
> announced on the [Project "a CQRS Journey"][cqrsjourneysite]" website. 
> If you would like to contribute to these bounded contexts or any other 
> aspect of the system, visit the [Project "a CQRS 
> Journey"][cqrsjourneysite] website or let us know at 
> [cqrsjourney@microsoft.com][cqrsemail]. 

本著で紹介するコンソト会議管理システムの最終リリースには採用しなかったものの、
ここで紹介したものの他に追加で3つの境界付けられたコンテキストに関する作業を行いました。
コミュニティのメンバーは、これらの機能やその他の機能に取り組んでおり、
本著に含まれないのリリースやアップデートは、[Project "a CQRS Journey"][cqrsjourneysite]のウェブサイトで発表されます。
これらの境界付けられたコンテキストやシステムの他の機能に貢献したい場合は、[Project "a CQRS Journey"][cqrsjourneysite]
のウェブサイトを参照するか、[cqrsjourney@microsoft.com][cqrsemail]に問い合わせてください。


> * **The Discounts Bounded Context:** This is a bounded context to handle
>   the process of managing and applying discounts to the purchase of
>   conference seats that would integrate with all three existing bounded
>   contexts.
> * **The Occasionally Disconnected Conference Management Client:** This
>   is a bounded context to handle management of conferences on-site with
>   functionality to handle label printing, recording attendee arrivals,
>   and additional seat sales.
> * **The Submissions and Schedule Management Bounded Context:** This is a
>   bounded context to handle paper submissions and conference event
>   scheduling written using Node.js.

* **割引のコンテキスト:** これは、会議席の購入時に割引を管理・適用するプロセスを
  処理するための境界付けられたコンテキストで、既存の3つの境界付けられたコンテキストに統合されます。
* **現場作業のコンテキスト:** この境界付けられたコンテキストは、ラベル印刷、参加者の到着記録、座席の追加販売などの機能を持ち、
  現場での会議管理を行います。
* **原稿投稿とスケジュール管理のコンテキスト:** 原稿の投稿と会議イベントのスケジューリングを行う
  境界付けられたコンテキストで、Node.jsを使って書かれています。

> **Note:** Wait listing is not implemented in this release, but members
> of the community are working on this and other features. Any
> out-of-band releases and updates will be announced on the [Project "a
> CQRS Journey"][cqrsjourneysite] website.

> **注:** 待機リストはこのリリースでは実装されていませんが、コミュニティのメンバーがこの機能やその他の機能に取り組んでいます。
> 本著にないリリースやアップデートは、[Project "a CQRS Journey"][cqrsjourneysite]のウェブサイトで発表されます。

> ## The Context Map for the Contoso Conference Management System

# コンソト会議管理システムのコンテキストマップ

> Figure 1 and the table that follows it represent a context map that 
> shows the relationships between the different bounded contexts that make 
> up the complete system, and as such it provides a high-level overview of 
> how the system is put together. Even though this context map appears to 
> be quite simple, the implementation of these bounded contexts, and more 
> importantly the interactions between them, are relatively sophisticated; 
> this enabled us to address a wide range of issues relating to the CQRS 
> pattern and event sourcing (ES), and provided a rich source from which 
> to capture many valuable lessons learned.

図1とその後の表は、システム全体を構成するさまざまな境界付けられたコンテキスト同士の関係を表すコンテキストマップです。
コンテキストマップは、システムがどのように構成されているかを示す高レベルの概要を示します。
このコンテキストマップは一見非常にシンプルに見えますが、これらの境界付けられたコンテキストの実装と
それらの間の相互作用は非常に高度です。特に、境界付けられたコンテキスト間の相互作用は、
CQRSパターンとイベントソーシング（ES）に関連する広範の課題と関係していて、
ここで得られる価値ある知見の源泉となります。

> **GaryPersona:** A frequent comment about CQRS projects is that it can
> be difficult to understand how all of the pieces fit together,
> especially if there a great many commands and events in the system.
> Often, you can perform some static analysis on the code to determine
> where events and commands are handled, but it is more difficult to
> automatically determine where they originate. At a high level, a
> context map can help you understand the integration between the
> different bounded contexts and the events involved. Maintaining
> up-to-date documentation about the commands and events can provide
> more detailed insight. Additionally, if you have tests that use
> commands as inputs and then check for events, you can examine the
> tests to understand the expected consequences of particular commands
> (see the section on testing in [Extending and Enhancing the Orders and
> Registrations Bounded Contexts][j_chapter4] for an example of this
> style of test).

> **Garyのペルソナ:** CQRSプロジェクトでよく言われるのが、システム内のコマンドやイベントが非常に多い場合に、
> それらをどのように組み合わせていくのかを理解しづらいということです。 多くの場合、コードの静的解析えば、
> イベントやコマンドがどこで処理されるのかを判断することはできますが、それらがどこから発生しているかを自動的に判断することは困難です。
> コンテキストマップは、異なる境界付けられたコンテキスト間の統合や、それらと関連するイベントを高いレベルで理解することに役立ちます。
> コマンドやイベントに関する最新のドキュメントを維持していけば、より詳細な洞察を得る手助けになります。
> さらに、コマンドを入力として使用したイベントをチェックするテストがあれば、そのテストを調べることで
> 特定のコマンドから予想される結果を理解することができます 。
> (このスタイルのテストの例については、[Extending and Enhancing the Orders and Registrations Bounded Contexts][j_chapter4]のテストに関するセクションを参照してください)

> Figure 1 shows the three bounded contexts that make up the Contoso 
> Conference Management System. The arrows in the diagram indicate the 
> flow of data as events between them. 

図1は、コンソト会議管理システムを構成する3つの境界付けられたコンテキストを示しています。
図中の矢印は、それらの間のイベントとしてのデータの流れを示しています。

![Figure 1][fig1]

> **Bounded contexts in the Contoso Conference Management System**

**コンソト会議管理システムの境界付けられたコンテキスト**

The following list provides more information about the arrows in figure 
1. You can find additional detail in the chapters that discuss the 
individual bounded contexts. 

以下のリストは、図1の矢印についての詳細です。個々の境界付けられたコンテキストについて説明している章ではさらに詳細を見ることができます。

> 1. Events that report when conferences have been created, updated, or
>    published. Events that report when seat types have been created or
>    updated.
> 2. Events that report when orders have been created or updated. Events
>    that report when Attendees have been assigned to seats.
> 3. Requests for a payment to be made.
> 4. Acknowledgement of the success or failure of the payment.

1. 会議が作成、更新、または公開されたことを報告するイベント。座席種類が作成または更新されたことを報告するイベント。
2. 2. 注文が作成または更新されたことを報告するイベント。出席者が座席に割り当てられたことを報告するイベント。
3. 3. 支払いを要求するイベント
4. 決済の成功または失敗の通知。

> **GaryPersona:** Some of the events that the conference management
> bounded context raises are coarse-grained and contain multiple fields.
> Remember that conference management is a create, read, update and
> delete (CRUD)-style bounded context and does not raise fine-grained
> domain-style events. For more information, see Chapter 5, "[Preparing
> for the V1 Release][j_chapter5]."

> **Garyのペルソナ:** 会議管理の境界付けられたコンテキストが発生させるいくつかは、粒度が荒く、複数のフィールドを含みます。
> カンファレンス管理はCRUD（create, read, update and delete）スタイルの境界付けられたコンテキストであり、
> 粒度の細かいドメインスタイルのイベントは発生しないことを覚えておいてください。詳しくは、第5章[V1リリースの準備][j_chapter5]をご覧ください。

> # Why did we choose these bounded contexts?

# これらの境界付けられたコンテキストを選択した理由

> During the planning stage of the journey, it became clear that these 
> were the natural divisions in the domain that could each contain their 
> own, independent domain models. Some of these divisions were easier to 
> identify than others. For example, it was clear early on that the 
> conference management bounded context is independent of the remainder of 
> the domain. It has clearly defined responsibilities that relate to 
> defining conferences and seat types and clearly defined points of 
> integration with the rest of the application. 

ここまでの旅の計画段階で、境界付けられたコンテキストが、それぞれ独立したドメインモデルを含むことができる
ドメインの自然な区分であるということを説明しました。これらの区分の中には、他の区分よりも簡単に特定できるものもありました。
例えば、会議管理のコンテキストは、ドメインの残りの部分から独立していることが早い段階でわかりました。
会議や座席種類の定義に関連する責任や、アプリケーションの他の部分との統合ポイントも明らかになりました。

> On the other hand, it took some time to realize that the orders and registrations bounded context is separate from the 
> payments bounded context. For example, it was not until the V2 release 
> of the application that all concepts relating to payments disappeared 
> from the Orders and Registrations bounded context when the 
> **OrderPaymentConfirmed** event became the **OrderConfirmed** event. 

一方で、注文と登録のコンテキストが、支払いのコンテキストとは別物であることがわかるのには時間がかかりました。
例えば、アプリケーションの V2 リリースまでは、 **OrderPaymentConfirmed** イベントが **OrderConfirmed** イベントになったことで、
支払いに関するすべての概念が注文と登録のコンテキストから消えていました。

> **GaryPersona:** We continued to refine the domain models right
> through the journey as our understanding of the domain deepened.

> **Garyのペルソナ:** 旅の中でドメインへの理解が深めながら、ドメインモデルを洗練させました。

> More practically, from the perspective of the journey, we wanted a set 
> of bounded contexts that would enable us to release a working 
> application with some core functionality and that would enable us to 
> explore a number of different implementation patterns: CQRS, CQRS/ES, as 
> well as integration with a legacy, CRUD-style bounded context.

より現実的には、旅という観点から、いくつかのコア機能を備えた実用的なアプリケーションをリリースすることができ、
多くの異なる実装パターン（CQRS、CQRS/ES、そしてレガシーなCRUDスタイルのバウンデッド・コンテキストとの統合）を
検討することができるような境界付けられたコンテキストを検討したいと考えていました。

> **BethPersona:** Contoso wants to release a usable application as
soon as possible, but be able to add both planned features and
customer-requested features as they are developed and with no
down time for the upgrades.

> **Bethのペルソナ:** コンソト社は、便利なアプリケーションをできるだけ早くリリースしたいと考えています。
> 一方で、予定していた機能と顧客からの要望の両方を追加開発しながら、アップグレード時のダウンタイムを発生させないようにしたいと考えています。

[j_chapter1]:     Journey_01_Introduction.markdown
[j_chapter4]:     Journey_04_ExtendingEnhancing.markdown
[j_chapter5]:     Journey_05_PaymentsBC.markdown
[r_chapter1]:     Reference_01_CQRSInContext.markdown
[cqrsemail]:      mailto:cqrsjourney@microsoft.com
[cqrsjourneysite]: http://cqrsjourney.github.com/
[fig1]:           images/Journey_02_BCs.png?raw=true