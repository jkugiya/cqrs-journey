### This version of this chapter was part of our working repository during the project. The final version of this chapter is now available on MSDN at [http://aka.ms/cqrs](http://aka.ms/cqrs).

> # Chapter 3: Orders and Registrations Bounded Context 

# 第3章: 注文と登録のコンテキスト

> _The first stop on our CQRS journey._

_CQRSを巡る旅の第一歩_

> "The Allegator is the same, as the Crocodile, and differs only in Name," John Lawson

> "アリゲーターとクロコダイルは名前が違うだけです。" John, Lawson

> # A description of the bounded context

# 境界付けられたコンテキスト

> The orders and registrations bounded context is partially responsible 
> for the booking process for attendees planning to come to a conference. 
> In the orders and registrations bounded context, a person (the 
> registrant) purchases seats at a particular conference. The registrant 
> also assigns names of attendees to the purchased seats (this is 
> described in chapter 5, [Preparing for the V1 Release][j_chapter5]). 

注文と登録のコンテキストは、カンファレンスへの参加を予定している参加者の予約プロセスに関する責務を部分的に負います。
注文と登録のコンテキストでは、登録者がカンファレンスの座席を購入します。
登録者はまた、購入した席に割り当てる出席者の名前を指定します（これについては、第5章の[V1リリースに向けて][j_chapter5]で説明します）。

> This was the first stop on our CQRS journey, so the team decided to 
> implement a core, but self-contained part of the system &mdash; orders and 
> registrations. The registration process must be as painless as possible 
> for attendees. The process must enable the business customer to ensure 
> that the maximum possible number of seats can be booked, and give them 
> the flexibility set the prices for the different seat types at a 
> conference. 

CQRSを巡る旅の第一歩は、チームがシステムの中核でありながら自己完結的なプロセスの「注文と登録」を実装することです。
登録プロセスは、参加者にとってできる限り負担のないものでなければなりません。

> Because this was the first bounded context addressed by the team, we 
> also implemented some infrastructure elements of the system to support 
> the domain's functionality. These included command and event message 
> buses and a persistence mechanism for aggregates. 

これはチームが初めて取り組んだ境界付けられたコンテキストなので、
このドメインの機能をサポートするためにシステムのインフラ要素も実装しました。
この中には、コマンドやイベントのメッセージバス、集約の永続化メカニズムなどがあります。

> **Note:** The Contoso Conference Management System described in this
> chapter is not the final version of the system. This guidance
> describes a journey, so some of the design decisions and
> implementation details change later in the journey. These
> changes are described in subsequent chapters.

> **注:** 本章のコンソト会議管理システムは、最終バージョンではありません。
> このガイダンスは旅の途中なので、中には設計上の決定事項や実装の詳細が旅の後半で変更されるものもあります。 
> 後続の章ではこれらの変更について述べます。

> Plans for enhancements to this bounded context in some future journey 
> include support for wait listing, whereby requests for seats are placed 
> on a wait list if there aren't sufficient seats available, and enabling 
> the business customer to set various types of discounts for seat types. 

この境界付けられたコンテキストでは、旅の中で予約待ちリストのサポートを機能追加することも計画しています。 この機能は、十分な座席がない場合に座席のリクエストを予約待ちリストに追加し、 顧客は座席タイプにさまざまな種類の割引を設定できます。 

> **Note:** Wait listing is not implemented in this release, but members
> of the community are working on this and other features. Any
> out-of-band releases and updates will be announced on the [Project "A
> CQRS Journey"][cqrsjourneysite] website.

> **注記:** このリリースでは予約待ちリストを実装していませんが、コミュニティのメンバーはこの機能や他の機能にも取り組んでいます。
> 本著にはないリリースやアップデートは、[CQRSを巡る旅][cqrsjourneysite]のウェブサイトで発表します。

> # Working definitions for this chapter

# この章における一時的定義

> This chapter uses a number of terms that we will define in a moment. For 
> more detail, and possible alternative definitions, see [A CQRS/ES Deep 
> Dive][r_chapter4] in the Reference Guide. 

本章では、いくつか一時的な用語の定義を行います。詳細や別の代替案については、リファレンスガイドの[CQRSとESについて深堀する][r_chapter4]を参照してください。

> **Command.** A _command_ is a request for the system to perform an action that changes the state of the system. Commands are imperatives; **MakeSeatReservation** is one example. In this bounded context, commands originate either from the UI as a result of a user initiating a request, or from a process manager when the process manager is directing an aggregate to perform an action.

**コマンド** _コマンド_ は、システムの状態を変化させる操作の実行を依頼するシステムへの要求です。コマンドの名称は命令形で、例えば**MakeSeatReservation**といった形をとります。この境界付けられたコンテキストでコマンドが発生するのは、UIからユーザーがリクエストを開始した結果としてUIから発生こともあれば、プロセスマネージャーからプロセスマネージャーが集約に操作を実行するよう指示するときに発生することもあります。

A single recipient processes a command. A command bus transports commands that command handlers then dispatch to aggregates. Sending a command is an asynchronous operation with no return value.

コマンドを受信するプロセスは1つだけです。コマンドバスがコマンドハンドラーからのコマンドの運び手となり、集約まで伝播します。コマンドの送信は戻り値を持たない非同期操作です。

> **GaryPersona:** For a discussion of some possible optimizations that also involve a slightly different definition of a command, see Chapter 6, [Versioning our System][j_chapter6].

> **Garyのペルソナ:** コマンドの捉え方を少々変えることで、少し異なった最適化を行うこともできますが、詳細は第6章の[システムのバージョン管理][j_chapter6]を参照してください。


> **Event.** An _event_, such as **OrderConfirmed**, describes something that has happened in the system, typically as a result of a command. Aggregates in the domain model raise events.
> Multiple subscribers can handle a specific event. Aggregates publish events to an event bus; handlers register for specific types of events on the event bus and then deliver the event to the subscriber. In this bounded context, the only subscriber is a process manager.

**イベント** **OrderConfirmed** といったイベントはシステム内で何かが発生したことを表します。
イベントは通常、コマンドの結果として発生します。 ドメインモデルの集約がイベントを発生させます。
イベントは複数の購読者が処理することがあります。集約がイベントをイベントバスに発行し、イベントハンドラがあらかじめ決まった種類のイベントを、
購読者に配信します。今回の例では、境界付けられたコンテキストにおける購読者はプロセスマネージャのみです。

> **Process manager.** In this bounded context, a _process manager_ is a class that coordinates the behavior of the aggregates in the domain. A process manager subscribes to the events that the aggregates raise, and then follow a simple set of rules to determine which command or commands to send. The process manager does not contain any business logic; it simply contains logic to determine the next command to send. The process manager is implemented as a state machine, so when it responds to an event, it can change its internal state in addition to sending a new command.
Our process manager is an implementation of the Process Manager pattern defined on pages 312 to 321 of the book by Gregor Hohpe and Bobby Woolf,  entitled _Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions_ (Addison-Wesley Professional, 2003).

**プロセスマネージャ** 今回の境界づけられたコンテキストでは、_プロセスマネージャ_ はドメイン内の集約の振る舞いを
調整するクラスです。プロセスマネージャは、集約で発生するイベントを購読し、単純なルールセットに従って、送信するコマンドを決定します。
プロセスマネージャはビジネスロジックを持たず、単に次に送信するコマンドを決定するロジックを持ちます。
プロセスマネージャはステートマシンでもあり、イベントを処理するとき、新しいコマンドを送信するだけでなく、
内部の状態を変更することもあります。

>> **MarkusPersona:** It can be difficult for someone new to the code to follow the flow of commands and events through the system. For a discussion of a technique that can help, see the section "Impact on testing" in Chapter 4, "[Extending and Enhancing the Orders and Registrations Bounded Contexts][j_chapter4]."

> **Markusのペルソナ** 初めてコードに触れる人にとって、システム内のコマンドやイベントの流れを追うのは難しいかもしれません。
> 第4章"[注文と登録の境界付けられたコンテキストの拡張と強化][j_chapter4]"の "テストへの影響 "のセクションで紹介するテクニックが参考になります。


> The process manager in this bounded context can receive commands as well 
> as subscribe to events. 

今回の境界付けられたコンテキストはコマンドの受信もイベントの購読も行います。

>> **GaryPersona:** The team initially referred to the process manager class in the orders bounded context as a saga. To find out why we decided to change the terminology, see the section [Patterns and Concepts](#patternsandconcepts) later in this chapter.

> **Garyのペルソナ** チームは当初、境界付けられたコンテキストにおけるプロセスマネージャをサーガと呼んでいた。
> この用語を変更した理由については、[パターンと概念](#patternsandconcepts)のセクションを参照してください。

> The Reference Guide contains additional definitions and explanations of 
> CQRS related terms.

このリファレンスガイドではCQRS関連用語に関する定義をさらに付け加えていきます。


> # Domain definitions (ubiquitous language)

# ドメイン定義 (普遍的な言語)

> The following list defines the key domain-related terms that the team used during the development of this Orders and Registrations bounded contexts.

以下のリストは、この「注文と登録」の境界づけられたコンテキストの開発中にチームが使用した重要なドメイン関連の用語を定義です。

> **Attendee.** An attendee is someone who is entitled to attend a conference. An Attendee can interact with the system to perform tasks such as manage his agenda, print his badge, and provide feedback after the conference. An attendee could also be a person who doesn't pay to attend a conference such as a volunteer, speaker, or someone with a 100% discount. An attendee may have multiple associated attendee types (speaker, student, volunteer, track chair, etc.)

**出席者（Attendee）** 出席者は、カンファレンスに出席する資格がある人を指します。出席者は、自分のアジェンダを管理したり、
バッジを印刷したり、カンファレンス後にフィードバックを提供するといったタスクを実行するためにシステムと対話することができます。
出席者の中には、ボランティア、スピーカー、または100%の割引を持つ人など、カンファレンスに参加料を支払わない人もいます。
出席者には、複数の関連する出席者の型（スピーカー、学生、ボランティア、トラックの議長など）があるかもしれません。

> **Registrant.** A registrant is a person who interacts with the system to place orders and to make payments for those orders. A registrant also creates the registrations associated with an order. A registrant may also be an attendee.

**登録者（Registrant）** 登録者は、注文を行い、それらの注文の支払いをするためにシステムと対話する人を指します。
登録者は、注文に関連づけられる登録も作成します。登録者は出席者である場合もあります。

> **User.** A user is a person such as an attendee, registrant, speaker, or volunteer who is associated with a conference. Each user has a unique record locator code that the user can use to access user-specific information in the system. For example, a registrant can use a record locator code to access her orders, and an attendee can use a record locator code to access his personalized conference agenda.

**ユーザー（User）** ユーザーは、カンファレンスに関連する出席者、登録者、スピーカー、またはボランティアのような人を指します。
各ユーザーには、システム内のユーザー固有の情報にアクセスするために使用できる一意の確認コードがあります。
たとえば、登録者は確認コードを使用して注文にアクセスすることができ、
出席者は確認コードを使用して自分のカンファレンスのアジェンダにアクセスすることができます。

> **CarlosPersona:** We intentionally implemented a record locator mechanism to return to a previously submitted order via the mechanism. This eliminates an often annoying requirement for users to create an account in the system and sign in in order to evaluate its usefulness. Our customers were adamant about this.

**Carlosのペルソナ** 私たちは以前に提出された注文に戻るための確認コードを使うというメカニズムは意図的な実装です。
これにより、システムの有用性を評価するためにユーザーがシステムにアカウントを作成し、サインインするという、
しばしば厄介な要件が排除できます。これは顧客の非常に強い要求です。

> **Seat assignment.** A seat assignment associates an attendee with a seat in a confirmed order. An order may have one or more seat assignments associated with it.

**席の割り当て（Seat assignment）** 席の割り当ては、出席者を確定した注文の席と関連付けます。
注文には、1つ以上の席の割り当てが関連付けられることがあります。

> **Order.** When a registrant interacts with the system, the system creates an order to manage the reservations, payment, and registrations. An order is confirmed when the registrant has successfully paid for the order items. An order contains one or more order items.

**注文（Order）** 登録者がシステムと対話すると、システムは登録を管理するための注文、支払い、および登録を作成します。
登録者が注文アイテムの支払いを正常に完了すると、注文が確定されます。注文には1つ以上の注文アイテムが含まれます。

> **Order item.** An order item represents a seat type and quantity, and is associated with an order. An order item exists in one of three states: created, reserved, or rejected. An order item is initially in the created state. An order item is in the reserved state if the system has reserved the quantity of seats of the seat type requested by the registrant. An order item is in the rejected state if the system cannot reserve the quantity of seats of the seat type requested by the registrant.

**注文アイテム（Order item）** 注文アイテムは、席の種類と数量を表し、注文に関連付けられます。
注文アイテムには、作成済、予約済、または拒否済という3つの状態があります。注文アイテムの初期状態は作成済です。
予約済は、システムが登録者が要求した席の種類の数量の席の予約が完了した状態です。
拒否済は、システムが登録者が要求した席の種類の数量の席を予約できなかった状態です。

> **Seat.** A seat represents the right to be admitted to a conference or to access a specific session at the conference such as a cocktail party, a tutorial, or a workshop. The business customer may change the quota of seats for each conference. The business customer may also change the quota of seats for each session.

**席（Seat）** 席は、カンファレンスに入場する権利、またはカンファレンスでの特定のセッション（カクテルパーティ、
チュートリアル、ワークショップなど）にアクセスする権利を表します。ビジネスの顧客は、
各カンファレンスの席の割り当てを変更することができます。ビジネスの顧客は、各セッションの席の割り当てを変更することもできます。

> Reservation. A reservation is a temporary reservation of one or more seats. The ordering process creates reservations. When a registrant begins the ordering process, the system makes reservations for the number of seats requested by the registrant. These seats are then not available for other registrants to reserve. The reservations are held for n minutes during which the registrant can complete the ordering process by making a payment for those seats. If the registrant does not pay for the seats within n minutes, the system cancels the reservation and the seats become available to other registrants to reserve.

**予約（Reservation）** 予約は1つ以上の席の一時的な予約を表します。注文プロセスは予約を作成します。
登録者が注文プロセスを開始すると、システムは登録者が要求した席の数の予約を行います。
これらの席は、他の登録者が予約することはできません。
予約は所定時間保持され、その間に登録者はこれらの席の支払いを行って注文プロセスを完了することができます。
登録者が所定時間以内に席の支払いをしない場合、システムは予約をキャンセルし、他の登録者が席を予約できるようになります。

> **Seat availability.** Every conference tracks seat availability for each type of seat. Initially, all of the seats are available to reserve and purchase. When a seat is reserved, the number of available seats of that type is decremented. If the system cancels the reservation, the number of available seats of that type is incremented. The business customer defines the initial number of each seat type to be made available; this is an attribute of a conference. A conference owner may adjust the numbers for the individual seat types.

**席の利用（Seat availability）.**
各カンファレンスは、各種類の席の利用を追跡します。最初は、すべての席が予約および購入可能です。
席を予約すると、予約した種類の席の利用可能な数が減少します。システムが予約をキャンセルすると、キャンセルした種類の利用可能な席の数が増加します。
ビジネスの顧客は、利用可能とする各席の種類の初期数を定義します。また、これはカンファレンスの属性とします。
カンファレンスのオーナーは、個々の席の種類の数を調整することができます。

> **Conference site.** You can access every conference defined in the system by using a unique URL. Registrants can begin the ordering process from this site.

**カンファレンスサイト（Conference site）**
システムで定義されたすべてのカンファレンスには、ユニークなURLを使用してアクセスできます。
登録者は、このサイトから注文プロセスを開始できます。


>> Each of the terms defined here was formulated through active discussions between the development team and the domain experts. The following is a sample conversation between developers and domain experts that illustrates how the team arrived at a definition of the term _attendee_.
> 
> ここで定義されている各用語は、開発チームとドメイン専門家の間での活発な議論を通じて形成されました。
> 以下は、チームが _出席者_ という用語の定義にたどり着いた方法を示す開発者とドメイン専門家の間のサンプル会話です。
>>
>> Developer 1: Here's an initial stab at a definition for _attendee_. "An attendee is someone who has paid to attend a conference. An attendee can interact with the system to perform tasks such as manage his agenda, print his badge, and provide feedback after the conference."
> 
> 開発者1: では、_出席者_ という用語の定義について考えたいと思います。
> 「出席者は、カンファレンスに参加するために支払いを行った人です。出席者は、自分のアジェンダを管理したり、
> バッジを印刷したり、カンファレンス後にフィードバックを提供するためのタスクを実行するためにシステムと対話できます。」
>>
>> Domain Expert 1: Not all attendees will pay to attend the conference. For example, some conferences will have volunteer helpers, also speakers typically don't pay. And, there may be some cases where an attendee gets a 100% discount.
>
> ドメイン専門家1: すべての出席者がカンファレンスに参加するために支払いを行うわけではありません。
> 例えば、いくつかのカンファレンスにはボランティアのヘルパーがいます。また、通常、スピーカーは支払いを行いません。
> 出席者が100%割引を受ける場合もあります。
> 
>>
>> Domain Expert 1: Don't forget that it's not the attendee who pays; that's done by the registrant.
>
> ドメイン専門家1: 出席者が支払いを行う人というわけではないことを忘れないでください。支払いを行うのは登録者です。
>>
>> Developer 1: So we need to say that Attendees are people who are authorized to attend a conference?
> 
> 開発者1: では、出席者とはカンファレンスに出席する認可を得た人たち、ということでしょうか？
> 
>>
>> Developer 2: We need to be careful about the choice of words here. The term authorized will make some people think of security and authentication and authorization.
> 
> 開発者2: ここでの言葉の選び方には注意が必要です。認可という言葉は、セキュリティや認証、認可を連想させるでしょう。
> 
>>
>> Developer 1: How about entitled?
> 
> 開発者1: それなら「権利がある」という言葉はどうでしょうか？
> 
>>
>> Domain Expert 1: When the system performs tasks such as printing badges, it will need to know what type of attendee the badge is for. For example, speaker, volunteer, paid attendee, and so on.
> 
> ドメイン専門家1: システムがバッジの印刷などのタスクを実行するとき、どの種類の出席者のバッジを印刷するのかを知る必要があります。
> 例えば、スピーカー、ボランティア、有料の出席者など。
>>
>> Developer 1: Now we have this as a definition that captures everything we've discussed. An attendee is someone who is entitled to attend a conference. An attendee can interact with the system to perform tasks such as manage his agenda, print his badge, and provide feedback after the conference. An attendee could also be a person who doesn't pay to attend a conference such as a volunteer, speaker, or someone with a 100% discount. An attendee may have multiple associated attendee types (speaker, student, volunteer, track chair, etc.)
>
> 開発者1: これまでの議論をすべてを全て加味した定義は以下となります。
> 出席者はカンファレンスに出席する権利がある人です。出席者は、自分のアジェンダを管理したり、
> バッジを印刷したり、カンファレンス後にフィードバックを提供するなどのタスクを実行するためにシステムと対話できます。
> 出席者は、ボランティアやスピーカー、100%の割引を受ける人など、カンファレンスに参加するために支払わない人もいます。
> 出席者は、関連する複数の出席者のタイプ（スピーカー、学生、ボランティア、トラックチェアなど）を持つことがあります。
> 

> # Requirements for creating orders

# 注文の作成に関する要求

> A registrant is the person who reserves and pays for (orders) seats at a conference. Ordering is a two-stage process: first, the registrant reserves a number of seats and then pays for the seats to confirm the reservation. If registrant does not complete the payment, the seat reservations expire after a fixed period and the system makes the seats available for other registrants to reserve.

登録者は、カンファレンスで席を予約し、支払いを行う（注文する）人です。注文には2つのプロセスがあります。まず、
登録者が席を予約し、次に席を確保するために支払いを行います。登録者が支払いを完了しない場合、席の予約は
一定期間後に失効し、システムは他の登録者が予約できるように席を利用可能にします。

> Figure 1 shows some of the early UI mockups that the team used to explore the seat-ordering story. 
 
図1は、チームが席注文のストーリーを探索するために使用した初期のUIモックアップの一部です。

![Figure 1][fig1]

> **Ordering UI mockups**

**注文UIのモックアップ**

> These UI mockups helped the team in several ways, allowing them to:

これらのUIモックアップは以下のように役立ちました。

> * Communicate the core team's vision for the system to the graphic designers who are on an independent team at a third-party company.
> * Communicate the domain expert's knowledge to the developers.
> * Refine the definition of terms in the ubiquitous language.
> * Explore "what if" questions about alternative scenarios and approaches.
> * Form the basis for the system's suite of acceptance tests.

* システムに対するコアチームのビジョンを、外注先の独立したチームに所属するグラフィックデザイナーに伝える
* ドメイン専門家の知識を開発者に伝える
* ユビキタス言語の用語の定義を精緻化する。
* 代替シナリオやアプローチに関する「もし...ならどうなるの？」という問いにこたえられるかどうかを探索する
* システムの受け入れテストの組み合わせの基礎を形成する。

> # Architecture

# アーキテクチャ

> The application is designed to deploy to Windows Azure. At this stage in the journey, the application consists of a web role that contains the ASP.NET MVC web application and a worker role that contains the message handlers and domain objects. The application uses SQL Database databases for data storage, both on the write-side and the read-side. The application uses the Windows Azure Service Bus to provide its messaging infrastructure.

これから紹介するアプリケーションはWindows Azureにデプロイするために設計されています。
この段階では、アプリケーションはASP.NET MVCのWebアプリケーションと、
メッセージハンドラやドメインオブジェクトを含むワーカーで構成されています。
アプリケーションはデータストレージとしてSQL Databaseデータベースを書き込み側と読み取り側の両方で使用します。
そのメッセージング基盤を提供するためにWindows Azure Service Busを使用しています。

> While you are exploring and testing the solution, you can run it locally, either using the Windows Azure compute emulator or by running the MVC web application directly and running a console application that hosts the handlers and domain objects. When you run the application locally, you can use a local SQL Server Express database instead of SQL Database, and use a simple messaging infrastructure implemented in a SQL Server Express database.

ソリューションを探索・テストするとき、Windows Azureの計算エミュレータを使用したり、
MVCのWebアプリケーションを直接実行したり、ハンドラとドメインオブジェクトをホストするコンソールアプリケーションを実行することで、
ローカル実行できます。
アプリケーションをローカルで実行する際には、SQL Databaseの代わりにローカルのSQL Server Expressを使用して、
SQL Server Expressで実装されたシンプルなメッセージング基盤を使用することができます。

> For more information about the options for running the application, see 
[Appendix 1][appendix1].

アプリケーションの実行オプションに関する詳細は、[付録1][appendix1]を参照してください

> **GaryPersona:** A frequently cited advantage of the CQRS pattern is that it enables you to scale the read side and write side of the application independently to support the different usage patterns. In this bounded context, however, the number of read operations from the UI is not likely to hugely out-number the write operations: this bounded context focuses on registrants creating orders. Therefore, the read side and the write side are deployed to the same Windows Azure worker role rather than to two separate worker roles that could be scaled independently.

**GaryPersona:** CQRSパターンでよく言及される利点は、アプリケーションの読み込み側と書き込み側を独立してスケールできるので、
両者の異なる利用パターンをそれぞれ別の方法でサポートできるということです。
しかし、今回の境界付けられたコンテキストでは、UIからの読み取り操作の数が書き込み操作の数を大幅に上回ることはありません。
今回の境界付けられたコンテキストは、注文を作成する登録者に焦点を当てます。
したがって、読み取り側と書き込み側は、独立してスケールする2つの別々のWindows Azureワーカーではなく、同じWindows Azureワーカーにデプロイします。


> # Patterns and concepts

# パターンと概念 <a name="patternsandconcepts"/>

> The team decided to implement the first bounded context without using 
> event sourcing in order to keep things simple. However, they did agree 
> that if they later decided that event sourcing would bring specific 
> benefits to this bounded context, then they would revisit this decision. 

チームは、物事をシンプルに保つために、イベントソーシングを使用せずに最初の境界付けられたコンテキストを実装することにしました。
ただし、後にイベントソーシングがこの境界付けられたコンテキストに具体的な利益をもたらすと判断した場合、
彼らはこの決定を再評価することにしました。

>> **Note** For a description of how event sourcing relates to the CQRS
>> pattern, see [Introducing Event Sourcing][r_chapter3] in the Reference
>> Guide.
> 
> **注** イベントソーシングとCQRSパターンの関係については、
> 参考ガイドの[イベントソーシングの紹介][r_chapter3]の説明を参照してください。

> One of the important discussions the team had concerned the choice of aggregates and entities that they would implement. The following images from the team's whiteboard illustrate some of their initial thoughts, and questions about the alternative approaches they could take with a simple conference seat reservation scenario to try and understand the pros and cons of alternative approaches.

チームが行った重要な議論の一つは、実装する集約とエンティティの選択に関してでした。
以下のチームのホワイトボードからの画像は、彼らの初期の考えや、代替的なアプローチの利点と欠点を理解するための
シンプルなカンファレンス席の予約シナリオに関する質問を示しています。

>> "A value I think developers would benefit greatly from recognizing is
>> the de-emphasis on the means and methods for persistence of objects in
>> terms of relational storage. Teach them to avoid modeling the domain
>> as if it was a relational store, and I think it will be easier to
>> introduce and understand both DDD and CQRS."  
>> &mdash; Josh Elster, CQRS Advisors Mail List
>
>開発者がオブジェクトの永続化の手段・方法に重きを置き過ぎてはならないことを理解することがとても重要です。
>DDDとCQRSの両方を導入し理解を手助けするには、開発者がドメインを関係データベースのように
>モデリングすることを避けるよう教えましょう。
> 
> &mdash; ジョッシュ・エルスター, CQRSアドバイザーズ メーリングリスト

>> **GaryPersona:** These diagrams deliberately exclude details of how
>> the system delivers commands and events through command and event
>> handlers. The diagrams focus on the logical relationships between the
>> aggregates in the domain.
> 
> **Garyのペルソナ** この図では、システムがコマンドやイベントのハンドラを通じてコマンドと
> イベントを配信する手段の詳細については、意図的に除外していいます。
> この図はドメイン内の集約間の論理的な関係に重点を置いています。

> This scenario considers what happens when a registrant tries to book
> several seats at a conference. The system must:
> - Check that sufficient seats are available.
> - Record details of the registration.
> - Update the total number of seats booked for the conference.
 
今回のシナリオでは、登録者がカンファレンスで複数の座席を予約しようとした場合のことを考えます。
システムは以下の要件を満たす必要があります。

- 予約時に空席があるかどうかを確認する
- 登録の詳細を記録する
- カンファレンスで予約された席の総数を記録する

>> **Note:** We deliberately kept the scenario simple to avoid
>> distractions while the team examines the alternatives. These examples
>> do not illustrate the final implementation of this bounded context. 
>
> **Note:** チームが大体シナリオを考える際により集中できるように意図的にシナリオを簡素にしています。
> これらの例は最終的な境界付けられたコンテキストの実装とは対応していません。

> The first approach considered by the team, shown in Figure 2, uses two 
> separate aggregates.

チームが初めに検討したのは、図2に示すような2つの別々の集約です。

![Figure 2][fig2]

> **Approach 1: Two separate aggregates**

**アプローチ1: 2つの別々の集約**

> The numbers in the diagram correspond to the following steps:
> 
> 1. The UI sends a command to register Attendees X and Y for
>    conference 157. The command is routed to a new **Order** aggregate.
> 2. The **Order** aggregate raises an event that reports that an order
>    has been created. The event is routed to the **SeatsAvailability**
>    aggregate.
> 3. The **SeatsAvailability** aggregate with an ID of 157 is
>    re-hydrated from the data store.
> 4. The **SeatsAvailability** aggregate updates its total
>    number of seats booked.
> 5. The updated version of the **SeatsAvailability**
>    aggregate is persisted to the data store.
> 6. The new **Order** aggregate, with an ID of 4239, is persisted to the
>    data store.

図の番号は以下のステップと対応しています。

1. UIは参加者XとYをカンファレンス157に登録するコマンドを送信する
2. **注文** 集約は注文が作られたことを示すイベントを発行する。イベントは **席利用** 集約が受信します。
3. データストアから **席利用** 集約を再構築します。
4. **席利用** 集約の予約席総数を更新します。
5. 更新した **席利用** 集約を永続化します。
6. 4239番のIDを持つ **注文** 集約をデータストアに永続化します。
   
>> **MarkusPersona:** The term rehydration refers to the process of
>> deserializing the aggregate instance from a data store.
> 
> **Markusのペルソナ** 再構築という用語の意味は、データストアから集約のインスタンスをデシリアライズすることです。
> 
>> **JanaPersona:** You could consider using the [Memento
>> pattern][memento] to handle the persistence and rehydration.
> 
> **Janaのペルソナ** 永続化と再構築の処理は [メメントパターン][memento] を使うといいかもしれません
> 

> The second approach considered by the team, shown in Figure 3, uses a 
single aggregate in place of two. 

チームが考えた2番目のアプローチは、図3に示すように、2つの集約ではなく、1つの集約を使用します。

![Figure 3][fig3]

> **Approach 2: A single aggregate**

**アプローチ2: 1つの集約**

> The numbers in the diagram correspond to the following steps:
> 
> 1. The UI sends a command to register Attendees X and Y onto
>    conference 157. The command is routed to the **Conference** aggregate
>    with an ID of 157.
> 2. The **Conference** aggregate with an ID of 157 is rehydrated from
>    the data store.
> 3. The **Order** entity validates the booking (it queries the 
>    **SeatsAvailability** entity to see if there are enough
>    seats left), and then invokes the method to update the number of
>    seats booked on the conference entity.
> 4. The **SeatsAvailability** entity updates its total number
>    of seats booked.
> 5. The updated version of the **Conference** aggregate is persisted to
>    the data store.

図の番号は以下のステップと対応しています。

1. UIは参加者XとYをカンファレンス157に登録するコマンドを送信する。
   コマンドはIDが157の **カンファレンス** 集約にルーティングされます。
2. IDが157の **カンファレンス** 集約をデータストアから再構築します。
3. **注文** エンティティは予約の有効性を検証し（**席利用** エンティティをクエリして、
   残りの席が十分あるかどうかを確認します）、その後、カンファレンスエンティティの予約席数を更新するメソッドを呼び出します。
4. **席利用** エンティティは予約席の総数を更新します。
5. 更新された **カンファレンス** 集約をデータストアに永続化します。

> The third approach considered by the team, shown in Figure 4, uses a 
> process manager to coordinate the interaction between two aggregates. 

チームが検討した3番目のアプローチは、図4のように、2つの集約間の相互作用を調整するプロセスマネージャを使用します。

![Figure 4][fig4]

> **Approach 3: Using a process manager**

**アプローチ3: プロセスマネージャを使用する**

> The numbers in the diagram correspond to the following steps:
> 
> 1. The UI sends a command to register Attendees X and Y for
>    conference 157. The command is routed to a new **Order** aggregate.
> 2. The new **Order** aggregate, with an ID of 4239,  is persisted to
>    the data store.
> 3. The **Order** aggregate raises an event that is handled by the
>    **RegistrationProcessManager** class.
> 4. The **RegistrationProcessManager** class determines that a command should
>    be sent to the **SeatsAvailability** aggregate with an ID of
>    157.
> 5. The **SeatsAvailability** aggregate is rehydrated from the
>    data store.
> 6. The total number of seats booked is updated in the
>    **SeatsAvailability** aggregate and it is persisted to the
>    data store.

図の番号は以下のステップと対応しています。

1. UIは参加者XとYをカンファレンス157に登録するコマンドを送信する。
   コマンドは新しい **注文** 集約にルーティングされます。
2. IDが4239の新しい **注文** 集約をデータストアに永続化します。
3. **注文** 集約がイベントを発行し、 **RegistrationProcessManager** がそれを処理します。
4. **RegistrationProcessManager** は、IDが157の **席利用** 集約にコマンドを送信する必要があると判断します。
5. **席利用** 集約をデータストアから再構築します。
6. **席利用** 集約の予約席の総数を更新し、データストアに永続化します。

>> **GaryPersona:** Process manager or saga? Initially the team referred to
>> the **RegistrationProcessManager** class as a saga. However, after they
>> reviewed the original definition of a saga from the paper
>> [Sagas][sagapaper] by Hector Garcia-Molina and Kenneth Salem, they
>> revised their decision. The key reasons for this are that the reservation
>> process does not include explicit compensation steps, and does not
>> need to be represented as a long-lived transaction.

> **Garyのペルソナ:** プロセスマネージャかサーガか？ 最初はチームは **RegistrationProcessManager** クラスをサーガと呼んでいました。
> しかし、Hector Garcia-MolinaとKenneth Salemによる論文 [Sagas][sagapaper] の元の定義を見直した後、彼らは決定を見直しました。
> その主な理由は、予約プロセスに明示的な補償ステップが含まれず、長寿命トランザクションとみなす必要がないためです。

> For more information about process managers and sagas, see chapter 6
> [A Saga on Sagas][r_chapter6] in the Reference Guide.

プロセスマネージャとサーガについての詳細は、参考ガイドの第6章 [サーガ・パターンにおけるサーガ][r_chapter6] を参照してください。
   
> The team identified the following questions about these approaches:
> 
> - Where does the validation that there are sufficient seats for the registration take place: in the **Order** or **SeatsAvailability** aggregate? 
> - Where are the transaction boundaries? 
> - How does this model deal with concurrency issues when multiple 
>   registrants try to place orders simultaneously? 
> - What are the aggregate roots?

以上を踏まえて、チームは以下の点について検討を行いました。

- 登録時に十分な席があるかどうかの検証はどこで行うべきか： **注文** 集約か **席利用** 集約か？
- トランザクションの境界をどこに置くべきか？
- 複数の登録者が同時に注文を行おうとした場合に、並行性に関する問題をどのように処理すべきか？
- 集約ルートは何であるべきか？

> The following sections discuss these questions in relation to the three
> approaches considered by the team.

次のセクションでは、チームが検討した3つのアプローチごとに、これらの疑問に答えていきます。

> ## Validation

## 入力チェック

> Before a registrant can reserve a seat, the system must check that there 
> are enough seats available. Although logic in the UI can attempt to 
> verify that there are sufficient seats available before it sends a 
> command, the business logic in the domain must also perform the check; 
> this is because the state may change between the time the UI performs
> the validation and the time that the system delivers the command to the
> aggregate in the domain.

登録者が席を予約する前に、システムは利用可能な席が十分にあるかどうかを確認する必要があります。
UIのロジックがコマンドを送信する前に十分な席が利用可能かどうかをチェックすることもできますが、
ドメインロジックでもチェックを行う必要があります。
これはUIがチェックを行う時とシステムが集約にコマンドを配信する時の間に状態が変わることがあるためです。

>> **JanaPersona:** When we talk about UI validation here, we are talking
>> about validation that the Model-View Controller (MVC) controller performs, not the browser.

> **Janaのペルソナ:** ここでのUIの入力チェックというのは、ブラウザではなく、Model-View Controller (MVC) コントローラで行うチェックのことです。

> In the first model, the validation must take place in either the 
> **Order** or **SeatsAvailability** aggregate. If it is the 
> former, the **Order** aggregate must discover the current seat 
> availability from the **SeatsAvailability** aggregate before 
> the reservation is made and before it raises the event. If it is the 
> latter, the **SeatsAvailability** aggregate must somehow notify the
> **Order** aggregate that it cannot reserve the seats, and that the
> **Order** aggregate must undo (or compensate for) any work that it has
> completed so far.

最初のモデルでは、検証は **注文** 集約と **席利用** 集約のどちらかで行う必要があります。
前者の場合、 **注文** 集約は予約が決定し、イベントを発行する前に **席利用** 集約から現在空いている席の情報を取得する必要があります。
後者の場合、**席利用** 集約は **注文** 集約に席を予約できないことを通知し、**注文** 集約がそれまでに完了した作業を元に戻す（または補償する）必要があります。

>> **BethPersona:** Undo is just one of many compensating actions that
>> occur in real life. The compensating actions could even be outside of
>> the system implementation and involve human actors: for example, a
>> Contoso clerk or the Business Customer calls the Registrant to tell
>> them that an error was made and that they should ignore the last
>> confirmation email they received from the Contoso system.

> **Bethのペルソナ:** 元に戻すことは、実際の生活で発生する多くの補償行動のうちの1つに過ぎません。
> 補償行動は、システムの実装外で、人間のアクターを巻き込むこともあります。
> 例えば、Contoso の事務員やビジネスカスタマーが登録者に電話をかけて、エラーが発生したことを伝え、
> Contoso システムから受け取った最後の確認メールを無視するように伝えてもよいでしょう。

> The second model behaves similarly, except that it is **Order** and 
> **SeatsAvailability** entities cooperating within a 
> **Conference** aggregate. 

2番目のモデルも同様に動作しますが、 **注文** エンティティと **席利用** エンティティが **カンファレンス** 集約内で協力できる点が異なります。

> In the third model, with the process manager, the aggregates exchange messages
> through the process manager about whether the registrant can make the reservation
> at the current time. 

プロセスマネージャを使用した3番目のモデルでは、集約はプロセスマネージャを介して、登録者が現在予約を行うことができるかどうかについてメッセージをやり取りします。

> All three models require entities to communicate about the validation 
> process, but the third model with the process manager appears more complex than the 
> other two. 

3つのモデルすべてで、エンティティは入力チェックのために通信する必要がありますが、プロセスマネージャを使用した3番目のモデルは他の2つよりも複雑に見えます。

> ## Transaction boundaries

## トランザクション境界

> An aggregate, in the DDD approach, represents a consistency boundary. 
> Therefore, the first model with two aggregates, and the third model with 
> two aggregates and a process manager will involve two transactions: one when the 
> system persists the new **Order** aggregate and one when the system 
> persists the updated **SeatsAvailability** aggregate.

DDDにおいては、集約は一貫性の境界を表します。
したがって、2つの集約を持つ最初のモデルと、2つの集約とプロセスマネージャを持つ3番目のモデルは、2つのトランザクションを必要とします。
1つの境界ははシステムが新しい **注文** 集約を永続化するときで、もう1つはシステムが更新した **席利用** 集約を永続化するときです。

>> **Note:** The term _consistency boundary_ refers to a boundary within
>> which you can assume that all the elements remain consistent with each other all the time.

> **注:** _トランザクション境界_ という用語は、すべての要素が常に互いに一貫していると仮定できる境界を指します。

> To ensure the consistency of the system when a registrant creates an 
> order, both transactions must succeed. To guarantee this, we must take 
> steps to ensure that the system is eventually consistent by ensuring 
> that the infrastructure reliably delivers messages to aggregates. 

登録者による注文作成時、システムの一貫性を確保するためには、両方のトランザクションが成功する必要があります。
これを保証するためには、インフラストラクチャがメッセージを集約に確実に配信することで、システムが結果整合性を持つようにする必要があります。

> In the second approach, which uses a single aggregate, we will only have a 
> single transaction when a registrant makes an order. This appears to be 
> the simplest approach of the three. 

1つの集約を使用する2番目のアプローチでは、登録者による注文作成時に1つのトランザクションのみが発生します。
これは3つのアプローチの中で最もシンプルなアプローチのように見えます。

> ## Concurrency

## 並行性

> The registration process takes place in a multi-user environment where many registrants could attempt to purchase seats simultaneously. The team decided to use the reservation pattern to address the concurrency issues in the registration process. In this scenario, this means that a registrant initially reserves seats (which are then unavailable to other registrants); if the registrant completes the payment within a timeout period, the system retains the reservation; otherwise the system cancels the reservation.

登録プロセスは沢山のユーザが存在する環境で行われるので、多くの登録者が同時に席を購入しようとする可能性があります。
チームは、登録プロセスの並行性問題に対処するために予約パターンを使用することにしました。
このシナリオでは、登録者は最初に席を予約し（その後他の登録者に利用できなくなります）、
登録者がタイムアウト期間内に支払いを完了すると、システムは予約を確保します。そうでない場合、システムは予約をキャンセルします。

> This reservation system introduces the need for additional message types; for example, an event to report that a registrant has made a payment, or report that a timeout has occurred.

この予約システムは、追加のメッセージタイプが必要となります。例えば、登録者が支払いを行ったことを報告するイベント、またはタイムアウトが発生したことを報告するイベントです。

> This timeout also requires the system to incorporate a timer somewhere 
> to track when reservations expire. 

このタイムアウトは、予約が期限切れになるタイミングを追跡するために、システムにタイマーを組み込む必要があります。

> Modeling this complex behavior with sequences of messages and the 
> requirement for a timer is best done using a process manager. 

メッセージの順序やタイマーといった複雑な要件をモデリングする場合、プロセスマネージャを用いるのが最適です。

> ## Aggregates and aggregate roots

## 集約と集約ルート

> In the two models that have the **Order** aggregate and the **SeatsAvailability** aggregate, the team easily identified the entities that make up the aggregate, and the aggregate root. The choice is not so clear in the model with a single aggregate: it does not seem natural to access orders through a **SeatsAvailability** entity, or to access the seat availability through an **Order** entity. Creating a new entity to act as an aggregate root seems necessary.

**注文** 集約と **席予約** 集約を持つ2つのモデルでは、チームは集約を構成するエンティティと集約ルートを簡単に特定できました。
1つの集約を持つモデルでは、注文を **席利用** エンティティを介して行ったり、席の予約を **注文** エンティティを介して行うことは自然ではないように見えます。
集約ルートとして機能する新しいエンティティを作成する必要がありそうです。

> The team decided on the model that incorporated a process manager because this 
> offers the best way to handle the concurrency requirements in this 
> bounded context. 

チームは、この境界付けられたコンテキストでの並行性要件を最適な方法を提供するため、プロセスマネージャを組み込んだモデルを選択しました。

> # Implementation details

# 実装の詳細

> This section describes some of the significant features of the orders 
> and registrations bounded context implementation. You may find it useful 
> to have a copy of the code so you can follow along. You can download it 
> from from the [Download center][downloadc], or check the evolution of 
> the code in the repository on github: 
> [mspnp/cqrs-journey-code][repourl]. 

このセクションでは、注文と登録の境界付けられたコンテキストの実装のいくつかの重要な機能について説明します。
コードを見ながら進めると理解しやすいかもしれません。コードは[ダウンロードセンター][downloadc]からダウンロードするか、
githubのリポジトリ [mspnp/cqrs-journey-code][repourl] でコードの進化を確認できます。

>> **Note:** Do not expect the code samples to match exactly the code in
>> the reference implementation. This chapter describes a step in the
>> CQRS journey, the implementation may well change as we learn more and
>> refactor the code.

> **注:** コードサンプルが参照実装のコードと完全に一致することは期待しないでください。
> この章はCQRSを巡る旅の一歩を説明しており、コードはより多くのことを学び、コードをリファクタリングしていくにつれて変更する可能性があります。

> ## High-level architecture

## 高レベルのアーキテクチャ

> As we described in the previous section, the team initially decided to 
> implement the reservations story in the Conference Management System 
> using the CQRS pattern but without using event sourcing. Figure 5 shows 
> the key elements of the implementation: an MVC web application, a data 
> store implemented using a Windows Azure SQL Database instance, the read and write models, and 
> some infrastructure components. 

前のセクションで説明したように、チームはまずはイベントソーシングを使用せずにCQRSパターンを適用したカンファレンス管理システムの予約ストーリーを実装することにしました。
図5は実装の主要な要素を示しており、MVC Webアプリケーション、Windows Azure SQL Databaseインスタンスを使用したデータストア、
リードモデル、ライトモデル、およびいくつかのインフラストラクチャコンポーネントが含まれています。

> **Note:** We'll describe what goes on inside the read and write models 
> later in this section. 

**注:** リードモデルとライトモデルの内部で何が行われるかについては、このセクションの後で説明します。

<!--
![Figure 5][fig5]
-->
![図5][fig5]

> **High-level architecture of the registrations bounded context**

**登録の境界付けられたコンテキストにおける高レベルアーキテクチャ**

> The following sections relate to the numbers in Figure 5 and provide 
> more detail about these elements of the architecture. 

以下のセクションの番号は図5と対応していて、アーキテクチャのこれらの要素について詳細を提供します。

> ### 1. Querying the read model

### 1. リードモデルへのクエリ

> The **ConferenceController** class includes an action named **Display** 
> that creates a view that contains information about a particular 
> conference. This controller class queries the read model using the 
> following code: 

**ConferenceController** クラスには、特定のカンファレンスに関する情報を含むビューを作成する **Display** というアクションがあります。
このコントローラクラスは、以下のコードを使用してリードモデルにクエリを発行します。

```Cs
public ActionResult Display(string conferenceCode)
{
	var conference = this.GetConference(conferenceCode);

	return View(conference);
}

private Conference.Web.Public.Models.Conference GetConference(string conferenceCode)
{
	var repo = this.repositoryFactory();
	using (repo as IDisposable)
	{
		var conference = repo.Query<Conference>().First(c => c.Code == conferenceCode);

		var conference =
			new Conference.Web.Public.Models.Conference { Code = conference.Code, Name = conference.Name, Description = conference.Description };

		return conference;
	}
}
```

> The read model retrieves the information from the data store and returns 
> it to the controller using a Data Transfer Object (DTO) class. 

リードモデルはデータストアから情報を取得し、データ転送オブジェクト(DTO)クラスを使用してコントローラに返します。

> ### 2. Issuing Commands

### 2. コマンドの発行

> The web application sends commands to the write model through a command 
> bus. This command bus is an infrastructure element that provides 
> reliable messaging. In this scenario, the bus delivers messages 
> asynchronously and once only to a single recipient.

Webアプリケーションはコマンドバスを介してライトモデルにコマンドを送信します。
コマンドバスというのは、信頼性のあるメッセージングを提供するインフラストラクチャ要素です。
このシナリオでは、コマンドバスはメッセージを非同期で、かつ一度だけ、単一の受信者に配信します。

> The **RegistrationController** class can send a **RegisterToConference** 
> command to the write model in response to user interaction. This command 
> sends a request to register one or more seats at the conference. The 
> **RegistrationController** class then polls the read model to discover 
> whether the registration request succeeded. See the section "6. Polling 
> the Read Model" below for more details.

ユーザの操作に応じて、**RegistrationController** クラスは **RegisterToConference** コマンドをライトモデルに送信できます。
このコマンドは、カンファレンスで1つ以上の席を登録するためのリクエストを送信します。
その後、**RegistrationController** クラスはリードモデルをポーリングして、登録リクエストが成功したかどうかを確認します。
詳細については、以下の「6. リードモデルのポーリング」セクションを参照してください。

> The following code sample shows how the **RegistrationController** sends
> a **RegisterToConference** command:

以下のコードサンプルでは、**RegistrationController** が **RegisterToConference** コマンドを送信しています。

```Cs
var viewModel = this.UpdateViewModel(conferenceCode, contentModel);

var command =
	new RegisterToConference
	{
		OrderId = viewModel.Id,
		ConferenceId = viewModel.ConferenceId,
		Seats = viewModel.Items.Select(x => new RegisterToConference.Seat { SeatTypeId = x.SeatTypeId, Quantity = x.Quantity }).ToList()
	};

this.commandBus.Send(command);
```

>> **Note:** All of the commands are sent asynchronously and do not 
> expect return values. 

**注:** すべてのコマンドは非同期で送信され、戻り値を期待しません。

> ### 3. Handling Commands

### 3. コマンドの処理

> Command handlers register with the command bus; the command bus can then 
> forward commands to the correct handler. 

コマンドハンドラをコマンドバスに登録すると、コマンドバスがコマンドを正しいハンドラに転送します。

> The **OrderCommandHandler** class handles the **RegisterToConference** 
> command sent from the UI. Typically, the handler is responsible for 
> initiating any business logic in the domain and for persisting any state 
> changes to the data store. 

**OrderCommandHandler**は、UIから送信された **RegisterToConference** コマンドを処理するクラスです。
通常、ハンドラにはドメイン内のビジネスロジックを開始し、データストアに状態変更を永続化する責任があります。

> The following code sample shows how the **OrderCommandHandler** 
> class handles the **RegisterToConference** command: 

以下のコードサンプルでは、**OrderCommandHandler** クラスが **RegisterToConference** コマンドを処理しています。

```Cs
public void Handle(RegisterToConference command)
{
    var repository = this.repositoryFactory();

    using (repository as IDisposable)
    {
        var seats = command.Seats.Select(t => new OrderItem(t.SeatTypeId, t.Quantity)).ToList();

        var order = new Order(command.OrderId, Guid.NewGuid(), command.ConferenceId, seats);

        repository.Save(order);
    }
}
```

> ### 4. Initiating business logic in the domain

### 4. ドメイン内のビジネスロジックの初期化

> In the previous code sample, the **OrderCommandHandler** class 
> creates a new **Order** instance. The **Order** entity is an aggregate 
> root, and its constructor contains code to initiate the domain logic. 
> See the section "Inside the Write Model" below for more details of what 
> actions this aggregate root performs. 

前のコードサンプルでは、**OrderCommandHandler** クラスが新しい **Order** インスタンスを作成しています。
**Order** エンティティは集約ルートであり、そのコンストラクタにはドメインロジックを初期化するコードが含まれています。
この集約ルートがどのようなアクションを実行するかの詳細については、以下の「ライトモデルの詳細」セクションを参照してください。

> ### 5. Persisting the changes

### 5. 変更の永続化

> In the previous code sample, the handler persists the new **Order** 
> aggregate by calling the **Save** method in the repository class. This 
> **Save** method also publishes any events raised by the **Order** 
> aggregate on the command bus. 

前のコードサンプルでは、ハンドラはリポジトリクラスの **Save** メソッドを呼び出すことで新しい **Order** 集約を永続化しています。
この **Save** メソッドは、**Order** 集約が発行したイベントをコマンドバスにも公開します。

> ### 6. Polling the read model

### 6. リードモデルのポーリング

> To provide feedback to the user, the UI must have a way to check whether 
> the **RegisterToConference** command succeeded. Like all commands in the 
> system, this command executes asynchronously and does not return a 
> result. The UI queries the read model to check whether the 
> command succeeded. 

ユーザにフィードバックを提供するためには、UIが **RegisterToConference** コマンドが成功したかどうかを確認する方法を持つことが必要です。
コマンドはシステム内のすべてのコマンドと同様に非同期で実行され、結果を返しません。
したがって、UIはリードモデルにクエリを発行して、コマンドが成功したかどうかを確認します。

> The following code sample shows the initial implementation where the 
> **RegistrationController** class polls the read model until either the 
> system creates the order or a timeout occurs. The **WaitUntilUpdated**
> method polls the read-model until it finds either that the order has
> been persisted or it times out.

以下のコードサンプルでは、**RegistrationController** クラスがリードモデルをポーリングして、
システムが注文を作成するか、タイムアウトが発生するまで待機しています。
**WaitUntilUpdated** メソッドで、注文が永続化されるか、タイムアウトするまでリードモデルをポーリングします。

```Cs
[HttpPost]
public ActionResult StartRegistration(string conferenceCode, OrderViewModel contentModel)
{
    ...
	
	this.commandBus.Send(command);

    var draftOrder = this.WaitUntilUpdated(viewModel.Id);

    if (draftOrder != null)
    {
        if (draftOrder.State == "Booked")
        {
            return RedirectToAction("SpecifyPaymentDetails", new { conferenceCode = conferenceCode, orderId = viewModel.Id });
        }
        else if (draftOrder.State == "Rejected")
        {
            return View("ReservationRejected", viewModel);
        }
    }

    return View("ReservationUnknown", viewModel);
}
```

> The team later replaced this mechanism for checking whether the system 
> saves the order with an implementation of the Post-Redirect-Get pattern. 
> The following code sample shows the new version of the 
> **StartRegistration** action method.

その後、チームは、システムが注文を保存したかどうかを確認するメカニズムを、Post-Redirect-Getパターンの実装に置き換えました。
以下のコードサンプルは、**StartRegistration** アクションメソッドの新しいバージョンです。

>> **Note:** For more information about the Post-Redirect-Get pattern see
>> the article [Post/Redirect/Get][prg] on Wikipedia.

**注:** Post-Redirect-Getパターンについての詳細は、Wikipediaの記事 [Post/Redirect/Get][prg] を参照してください。

```Cs
[HttpPost]
public ActionResult StartRegistration(string conferenceCode, OrderViewModel contentModel)
{
    ...

    this.commandBus.Send(command);

    return RedirectToAction("SpecifyRegistrantDetails", new { conferenceCode = conferenceCode, orderId = command.Id });
}
```

> The action method now redirects to the **SpecifyRegistrantDetails** view 
> immediately after it sends the command. The following code sample shows 
> how the **SpecifyRegistrantDetails** action polls for the order in the 
> repository before returning a view. 

アクションメソッドは、コマンドを送信した直後にすぐに **SpecifyRegistrantDetails** ビューにリダイレクトします。
以下のコードサンプルでは、ビューを返す前に **SpecifyRegistrantDetails** アクションがリポジトリで注文をポーリングしています。

```Cs
[HttpGet]
public ActionResult SpecifyRegistrantDetails(string conferenceCode, Guid orderId)
{
    var draftOrder = this.WaitUntilUpdated(orderId);
    
	...
}
```

> The advantages of this second approach, using the Post-Redirect-Get 
> pattern instead of in the **StartRegistration** post action are that it 
> works better with the browser's forward and back navigation buttons, and 
> that it gives the infrastructure more time to process the command before 
> the MVC controller starts polling. 

**StartRegistration** ポストアクションではなく、Post-Redirect-Getパターンを使用するこの2番目のアプローチの利点は、
ブラウザの進むボタンと戻るボタンとの互換性が向上し、MVCコントローラがポーリングを開始する前にインフラストラクチャにコマンドを処理する時間を与えられることです。


> ## Inside the write model

## ライトモデルの詳細

> ### Aggregates

### 集約

> The following code sample shows the **Order** aggregate.

以下は **Order** 集約のサンプルです。

```Cs
public class Order : IAggregateRoot, IEventPublisher
{
    public static class States
    {
        public const int Created = 0;
        public const int Booked = 1;
        public const int Rejected = 2;
        public const int Confirmed = 3;
    }

    private List<IEvent> events = new List<IEvent>();

    ...

    public Guid Id { get; private set; }

    public Guid UserId { get; private set; }

    public Guid ConferenceId { get; private set; }

    public virtual ObservableCollection<TicketOrderLine> Lines { get; private set; }

    public int State { get; private set; }

    public IEnumerable<IEvent> Events
    {
        get { return this.events; }
    }

    public void MarkAsBooked()
    {
        if (this.State != States.Created)
            throw new InvalidOperationException();

        this.State = States.Booked;
    }

    public void Reject()
    {
        if (this.State != States.Created)
            throw new InvalidOperationException();

        this.State = States.Rejected;
    }
}
```

> Notice how the properties of the class are not virtual. In the original 
> version of this class, the properties **Id**, **UserId**, 
> **ConferenceId**, and **State** were all marked as virtual. The 
> following conversation between two developers explores this decision. 

クラスのプロパティがvirtualではないことに注意してください。
このクラスの元のバージョンでは、プロパティ **Id**、**UserId**、**ConferenceId**、**State** がすべてvirtualとしてマークされていました。
以下の会話は、この決定について2人の開発者が探求しているものです。

>> *Developer 1:* I'm really convinced you should not make the 
>> property virtual, except if required by the object-relational mapping (ORM) layer. If this is just for 
>> testing purposes, entities and aggregate roots should never be tested 
>> using mocking. If you need mocking to test your entities, this is a 
>> clear smell that something is wrong in the design. 

> *開発者1: * 私は、オブジェクトリレーショナルマッピング(ORM)レイヤーによって必要とされていない限り、プロパティをvirtualにするべきではないと思います。
> テスト目的かもしれませんが、エンティティや集約ルートモックを使用してテストすべきではありません。
> エンティティをテストするためにモックが必要なのは、明確なテストスメルで、設計に何か問題があります。

>> *Developer 2:* I prefer to be open and extensible by default. You 
>> never know what needs may arise in the future, and making things 
>> virtual is hardly a cost. This is certainly controversial and a bit 
>> non-standard in .NET, but I think it's OK. We may only need virtuals 
>> on lazy-loaded collections. 

> *開発者2: * 私は、デフォルトでオープンで拡張可能である方が好みです。
> 将来どのようなニーズが発生するかはわかりませんし、virtualにすることはほとんどコストがかかりません。
> これは確かに議論の余地があり、.NET において少し非標準的ですが、私は大丈夫だと思います。
> 遅延ロードされたコレクションだけでvirtualが必要になるかもしれません。

>> *Developer 1:* Since CQRS usually makes the need for lazy load 
>> vanish, you should not need it either. This leads to even simpler code. 

> *開発者1: * CQRSは通常、遅延ロードの必要性をなくすので、それも必要ないはずです。
> virtual を無くすことで、さらにシンプルなコードになります。

>> *Developer 2:* CQRS does not dictate usage of event sourcing (ES), so if you're 
>> using an aggregate root that contains an object graph, you'd need that 
>> anyway, right? 

> *開発者2: * CQRSでは必ずしもイベントソーシング(ES)を行うわけではないので、オブジェクトグラフを含む集約ルートを使用している場合、
> 遅延ロードが必要になるのでは？

>> *Developer 1:* This is not about ES, it's about DDD. When your 
>> aggregate boundaries are right, you don't need delay loading. 

> *開発者1: * これはESの話ではなく、DDDの話です。
> 集約の境界を正しく設計すれば、遅延ロードは必要ありません。

>> *Developer 2:* To be clear, the aggregate boundary is here to group 
>> things that should change together for reasons of consistency. A lazy 
>> load would indicate that things that have been grouped together don't 
>> really need this grouping.

> *開発者2: * あなたが言いたいことは、集約の境界はというのは、一貫性のために一緒に変更するべきものをグループ化したもので、
> 遅延ロードが必要ということは、グループ化したものが実際にはこのグループ化を必要としていないことを示している、ということですね？

>> *Developer 1:* I agree. I have found that lazy-loading in the 
>> command side means I have it modeled wrong. If I don't need the value 
>> in the command side, then it shouldn't be there. In addition, I 
>> dislike virtuals unless they have an intended purpose (or some 
>> artificial requirement from an object-relational mapping (ORM) tool). In my opinion, it violates the 
>> Open-Closed principle: you have opened yourself up for modification in 
>> a variety of ways that may or may not be intended and where the 
>> repercussions might not be immediately discoverable, if at all. 

> *開発者1: * そういうことです。コマンドサイドでの遅延ロードは、モデルが間違っていることを示していると考えています。
> コマンドサイドで必要ない値は、その集約に含むべきではありません。
> また、意図した目的（またはオブジェクトリレーショナルマッピング(ORM)ツールからの人工的な要件）がない限り、私はvirtualを使うことを好まないです。
> これはオープン・クローズドの原則に違反していると思います。意図しない変更の可能性があり、その影響がすぐにはわからない場合があるため、
> あらゆる方法で変更を受け入れるように自分自身を開放してしまっているからです。

>> *Developer 2:* Our **Order** aggregate in the model has a list of 
>> **Order Items**. Surely we don't need to load the lines to mark it as 
>> Booked? Do we have it modeled wrong there? 

> *開発者2: * モデルの **Order** 集約には **Order Items** のリストがあります。
> それを Booked(予約済) としてマークするためにロードする必要はないでしょうか？

>> *Developer 1:* Is the list of **Order Items** that long? If it is, 
>> the modeling may be wrong because you don't necessarily need 
>> transactionality at that level. Often, doing a late round trip to get 
>> and updated **Order Items** can be more costly that loading them 
>> up front: you should evaluate the usual size of the collection and do 
>> some performance measurement. Make it simple first, optimize if 
>> needed. 

> *開発者1: * **Order Items** のリストはそんなに大きなものになるでしょうか？もしそうなら、モデリングが間違っているかもしれません。
> そのレベルでトランザクション性が必要とは限らないからです。
> **Order Items** を取得してから更新するという遅いラウンドトリップは、あらかじめロードするよりもコストがかかることがよくあります。
> コレクションの通常のサイズを評価し、パフォーマンス測定を行うべきです。
> まずは単純にしておき、最適化は必要に応じて行いましょう。

> &mdash; *Thanks to J&eacute;r&eacute;mie Chassaing and Craig Wilson*



> ### Aggregates and process managers

### 集約とプロセスマネージャ

> Figure 6 shows the entities that exist in the write-side model. There 
> are two aggregates, **Order** and **SeatsAvailability**, each 
> one containing multiple entity types. Also there is a 
> **RegistrationProcessManager** class to manage the interaction between the
> aggregates. 

図6は、ライトサイドモデルのエンティティです。
**Order** と **SeatsAvailability** という2つの集約は、それぞれ複数のエンティティを含んでいます。

> The table in the Figure 6 shows how the process manager behaves given a current 
> state and a particular type of incoming message. 

図6の表は、現在の状態と受信したメッセージの種類に応じて、プロせずマネージャがどのように振る舞うかを示しています。

<!--
![Figure 6][fig6]
-->
![図6][fig6]

> **Domain objects in the write model**

**ライトモデルのドメインオブジェクト**

> The process of registering for a conference begins when the UI sends a 
> **RegisterToConference** command. The infrastructure delivers this 
> command to the **Order** aggregate. The result of this command is that 
> the system creates a new **Order** instance, and that the new **Order** 
> instance raises an **OrderPlaced** event. The following code sample from 
> the constructor in the **Order** class shows this happening. Notice how 
> the system uses GUIDs to identify the different entities. 

UIが **RegisterToConference** コマンドを送信すると、カンファレンスへの登録プロセスが開始します。

```Cs
public Order(Guid id, Guid userId, Guid conferenceId, IEnumerable<OrderItem> items)
{
    this.Id = id;
    this.UserId = userId;
    this.ConferenceId = conferenceId;
    this.Lines = new ObservableCollection<OrderItem>(items);

    this.events.Add(
        new OrderPlaced
        {
            OrderId = this.Id,
            ConferenceId = this.ConferenceId,
            UserId = this.UserId,
            Seats = this.Lines.Select(x => new OrderPlaced.Seat { SeatTypeId = x.SeatTypeId, Quantity = x.Quantity }).ToArray()
        });
}
```

>> **Note:** To see how the infrastructure elements deliver commands and
>  events, see Figure 7.

**注:** インフラストラクチャ要素がどのようにコマンドとイベントを配信するかについては、図7を参照してください。

> The system creates a new **RegistrationProcessManager** instance to manage the 
> new order. The following code sample from the **RegistrationProcessManager** 
> class shows how the process manager handles the event. 

システムは新しい **RegistrationProcessManager** インスタンスを作成して、新しい注文を管理します。
次は **RegistrationProcessManager** クラスのサンプルコードで、プロセスマネージャがイベントを処理する方法を示しています。

```Cs
public void Handle(OrderPlaced message)
{
    if (this.State == ProcessState.NotStarted)
    {
        this.OrderId = message.OrderId;
        this.ReservationId = Guid.NewGuid();
        this.State = ProcessState.AwaitingReservationConfirmation;

        this.AddCommand(
            new MakeSeatReservation
            {
                ConferenceId = message.ConferenceId,
                ReservationId = this.ReservationId,
                NumberOfSeats = message.Items.Sum(x => x.Quantity)
            });
    }
    else
    {
        throw new InvalidOperationException();
    }
}
```

> The code sample shows how the process manager changes its state and sends a new 
> **MakeSeatReservation** command that the 
> **SeatsAvailability** aggregate handles. The code sample also 
> illustrates how the process manager is implemented as a state machine that receives 
> messages, changes its state, and sends new messages. 

このサンプルコードでは、プロセスマネージャが状態を変更し、**SeatsAvailability** 集約が処理する新しい **MakeSeatReservation** コマンドを送信しています。
このプロセスマネージャは、メッセージを受け取って状態を変更し、新しいメッセージを送信するステートマシンとして実装されています。

>> **MarkusPersona:** Notice how we generate a new globally unique identifier (GUID) to identify the new reservation. We use these GUIDs to correlate messages to the correct process manager and aggregate instances.

> **Markusのペルソナ:** 新しい予約を識別するために新しいグローバルユニーク識別子(GUID)を生成していることに注意してください。

> When the **SeatsAvailability** aggregate receives a 
**MakeReservation** command, it makes a reservation if there are enough 
available seats. The following code sample shows how the 
**SeatsAvailability** class raises different events depending 
on whether or not there are sufficient seats. 

**SeatsAvailability** 集約は **MakeReservation** コマンドを受け取ると、利用可能な席があれば予約を行います。
次のサンプルコードでは、**SeatsAvailability** クラスが利用可能な席があるかどうかに応じて異なるイベントを発生させています。


```Cs
public void MakeReservation(Guid reservationId, int numberOfSeats)
{
    if (numberOfSeats > this.RemainingSeats)
    {
        this.events.Add(new ReservationRejected { ReservationId = reservationId, ConferenceId = this.Id });
    }
    else
    {
        this.PendingReservations.Add(new Reservation(reservationId, numberOfSeats));
        this.RemainingSeats -= numberOfSeats;
        this.events.Add(new ReservationAccepted { ReservationId = reservationId, ConferenceId = this.Id });
    }
}
```

> The **RegistrationProcessManager** class handles the 
**ReservationAccepted** and **ReservationRejected** events. This 
reservation is a temporary reservation for seats to give the user the 
opportunity to make a payment. The process manager is responsible for releasing 
the reservation when either the purchase is complete, or the reservation 
timeout period expires. The following code sample shows how the process manager 
handles these two messages.

**RegistrationProcessManager** は **ReservationAccepted** と **ReservationRejected** というイベントを処理します。
この時の予約は、ユーザが支払いを行う時のための席の一時的な予約です。
プロセスマネージャは、購入完了時、もしくは予約のタイムアウト時に予約を解放する責任があります。
次のサンプルコードでは、プロセスマネージャがこれらの2つのイベントを処理しています。

```Cs
public void Handle(ReservationAccepted message)
{
    if (this.State == ProcessState.AwaitingReservationConfirmation)
    {
        this.State = ProcessState.AwaitingPayment;

        this.AddCommand(new MarkOrderAsBooked { OrderId = this.OrderId });
        this.commands.Add(
            new Envelope<ICommand>(new ExpireOrder { OrderId = this.OrderId, ConferenceId = message.ConferenceId })
            {
                Delay = TimeSpan.FromMinutes(15),
            });
    }
    else
    {
        throw new InvalidOperationException();
    }
}

public void Handle(ReservationRejected message)
{
    if (this.State == ProcessState.AwaitingReservationConfirmation)
    {
        this.State = ProcessState.Completed;
        this.AddCommand(new RejectOrder { OrderId = this.OrderId });
    }
    else
    {
        throw new InvalidOperationException();
    }
}
```

> If the reservation is accepted, the process manager starts a timer running by 
> sending an **ExpireOrder** command to itself, and sends a 
> **MarkOrderAsBooked** command to the **Order** aggregate. Otherwise, it 
> sends a **ReservationRejected** message back to the **Order** aggregate. 

予約が受け入れられた場合、プロセスマネージャは **ExpireOrder** コマンドを自身に送信するタイマーを開始し、
**Order** 集約に **MarkOrderAsBooked** コマンドを送信します。
それ以外の場合は、**Order** 集約に **ReservationRejected** メッセージを送信します。

> The previous code sample shows how the process manager sends the 
> **ExpireOrder** command. The infrastructure is responsible for 
> holding the message in a queue for the delay of fifteen minutes. 

前のサンプルコードでは、プロセスマネージャが **ExpireOrder** コマンドを送信する方法を示しています。
インフラストラクチャが、15分後に送信するためにメッセージをキューに保持する責任を持ちます。

> You can examine the code in the **Order**, 
> **SeatsAvailability**, and **RegistrationProcessManager** classes 
> to see how the other message handlers are implemented. They all follow 
> the same pattern: receive a message, perform some logic, and send a 
> message. 

**Order**、**SeatsAvailability**、**RegistrationProcessManager** クラスのコードを見れば、
他のメッセージハンドラの実装方法を確認できます。
これらはすべて、メッセージを受け取り、いくつかのロジックを実行し、メッセージを送信するという同じパターンに従っています。


> **JanaPersona:** The code samples shown in this chapter are from an
> early version of the Conference Management System. The next chapter
> shows how the design and implementation evolved as the team explored
> the domain and learned more about the CQRS pattern.

**Janaのペルソナ:** この章のコードサンプルは、カンファレンス管理システムの初期バージョンです。
次の章では、チームがドメインを探求し、CQRSパターンについてより多くのことを学ぶことで、設計と実装の進化にどのように影響をもたらすかを示します。

> ### Infrastructure

### インフラストラクチャ

> The sequence diagram in Figure 7 shows how the infrastructure elements 
> interact with the domain objects to deliver messages. 

図7のシーケンス図は、インフラストラクチャ要素がメッセージを配信するためにドメインオブジェクトとどのようにやり取りするかを示しています。

![Figure 7][fig7]

> **Infrastructure sequence diagram**

**インフラストラクチャシーケンス図**

> A typical interaction begins when an MVC controller in the UI sends a 
> message using the command bus. The message sender invokes the **Send** 
> method on the command bus asynchronously. The command bus then stores 
> the message until the message recipient retrieves the message and 
> forwards it to the appropriate handler. The system includes a number of 
> command handlers that register with the command bus to handle specific 
> types of commands. For example, the **OrderCommandHandler** class defines 
> handler methods for the **RegisterToConference**, **MarkOrderAsBooked**, 
> and **RejectOrder** commands. The following code sample shows the 
> handler method for the **MarkOrderAsBooked** command. Handler methods 
> are responsible for locating the correct aggregate instance, calling 
> methods on that instance, and then saving that instance. 

典型的な対話は、UIのMVCコントローラがコマンドバスを通じてメッセージを送信して始まります。
メッセージ送信者(UI)は、コマンドバスの **Send** メソッドを非同期で呼び出します。
その後、コマンドバスは、メッセージ受信者がメッセージを取得して適切なハンドラに転送するまでメッセージを保持します。
システムには、特定のコマンドを処理する沢山のコマンドハンドラがあり、それらはコマンドバスに登録されています。
たとえば、**OrderCommandHandler** クラスは、**RegisterToConference**、**MarkOrderAsBooked**、**RejectOrder** といったコマンドのためのハンドラメソッドを持ちます。
次のサンプルコードは、**MarkOrderAsBooked** コマンドのハンドラメソッドです。
このハンドラメソッドの責務は、正しい集約インスタンスを見つけ、そのインスタンスのメソッドを呼び出し、そのインスタンスを保存することです。


```Cs
public void Handle(MarkOrderAsBooked command)
{
    var repository = this.repositoryFactory();

    using (repository as IDisposable)
    {
        var order = repository.Find<Order>(command.OrderId);

        if (order != null)
        {
            order.MarkAsBooked();
            repository.Save(order);
        }
    }
}
```

The class that implements the **IRepository** interface is responsible 
for persisting the aggregate and publishing any events raised by the 
aggregate on the event bus, all as part of a transaction. 

**CarlosPersona:** The team later discovered an issue with this when 
they tried to use Windows Azure Service Bus as the messaging 
infrastructure. Windows Azure Service Bus does not support distributed 
transactions with databases. For a discussion of this issue, see 
[Preparing for the V1 Release][j_chapter5] later in this guide. 

The only event subscriber in the reservations bounded context is the 
**RegistrationProcessManager** class. Its router subscribes to the event bus to 
handle specific events, as shown in the following code sample from the 
**RegistrationProcessManager** class. 

> **Note:** We use the term handler to refer to the classes that handle 
> commands and events and forward them to aggregate instances, and the
> term router to refer to the classes that handle events and commands 
> and forward them to process manager instances. 

```Cs
public void Handle(ReservationAccepted @event)
{
	var repo = this.repositoryFactory.Invoke();
	using (repo as IDisposable)
	{
        lock (lockObject)
        {
            var process = repo.Find<RegistrationProcessManager>(@event.ReservationId);
            process.Handle(@event);

            repo.Save(process);
        }
	}
}
```

Typically, an event handler method loads a process manager instance, passes the 
event to the process manager, and then persists the process manager instance. In this 
case, the **IRepository** instance is responsible for persisting the 
process manager instance and for sending any commands from the process manager 
instance to the command bus. 

## Using the Windows Azure Service Bus

To transport Command and Event messages, the team decided to use the 
Windows Azure Service Bus to provide the low-level messaging 
infrastructure. This section describes how the system uses the Windows 
Azure Service Bus and some of the alternatives and trade-offs the team 
considered during the design phase. 

> **JanaPersona:** The team at Contoso decided to use the Windows Azure
> Service Bus because it offers out-of-the-box support for the messaging
> scenarios in the Conference Management System. This minimizes the
> amount of code that the team needs to write, and provides for a
> robust, scalable messaging infrastructure. The team plans to use
> features such as duplicate message detection and guaranteed message
> ordering. For a summary of the differences between Windows Azure
> Service Bus and Windows Azure Queues, see [Windows Azure Queues and 
> Windows Azure Service Bus Queues - Compared and Contrasted][sbq] on
> MSDN.

Figure 8 shows how both command and event messages flow through the system.
MVC controllers in the UI and domain objects use **CommandBus** 
and **EventBus** instances to send **BrokeredMessage** messages to one 
of the two topics in the Windows Azure Service Bus. To receive messages, 
the handler classes register with the **CommandProcessor** and 
**EventProcessor** instances that retrieve messages from the topics by 
using the **SubscriptionReceiver** class. The **CommandProcessor** class 
determines which single handler should receive a command message; the 
**EventProcessor** class determines which handlers should receive an 
event message. The handler instances are responsible for invoking 
methods on the domain objects. 

> **Note:** A Windows Azure Service Bus topic can have multiple 
> subscribers. The Windows Azure Service Bus delivers messages sent to a 
> topic to all its subscribers. Therefore, one message can have multiple 
> recipients. 

![Figure 8][fig8]

**Message flows through a Windows Azure Service Bus topic**

In the initial implementation, the **CommandBus** and **EventBus** 
classes are very similar. The only difference between the **Send** 
method and the **Publish** method is that the **Send** method expects 
the message to be wrapped in an **Envelope** class. The **Envelope** 
class enables the sender to specify a time delay for the message 
delivery. 

Events can have multiple recipients. In the example shown 
in Figure 8, the **ReservationRejected** event is sent to the 
**RegistrationProcessManager**, the **WaitListProcessManager**, and one other 
destination. The **EventProcessor** class identifies the list of
handlers to receive the event by examining its list of registered
handlers.

A command has only one recipient. In Figure 8, the 
**MakeSeatReservation** is sent to the **SeatsAvaialbility** aggregate. 
There is just a single handler registered for this subscription. The
**CommandProcessor** class identifies the handler to receive the command
by examining its list of registered handlers.

This implementation gives rise to a number of questions:

1. How do you limit delivery of a command to a single recipient?
2. Why have separate **CommandBus** and **EventBus** classes if they are
   so similar?
3. How scalable is this approach?
4. How robust is this approach?
5. What is the granularity of a topic and a subscription?
6. How are commands and events serialized?

The following sections discuss these questions.

### Delivering a command to a single recipient

This discussion assumes you that you have a basic understanding of the 
differences between Windows Azure Service Bus queues and topics. For an 
introduction to Windows Azure Service Bus, see [Technologies Used in the 
Reference Implementation][r_chapter9] in the Reference Guide. 

With the implementation shown in Figure 8, two things are necessary to 
ensure that a single handler handles a command message. First, 
there should only be a single subscription to the 
**conference/commands** topic in Windows Azure Service Bus; remember 
that a Windows Azure Service Bus topic may have multiple subscribers. 
Second, the **CommandProcessor** should invoke a single handler for each 
command message that it receives. There is no way in Windows Azure
Service Bus to restrict a topic to a single subscription; therefore, the
developers must be careful to create just a single subscription on a
topic that is delivering commands. 

> **GaryPersona:** A separate issue is to ensure that the handler
> retrieves commands from the topic and processes them only once. You
> must ensure either that the command is idempotent, or that the system
> guarantees to process the command only once. The team will address
> this issue in a later stage of the journey. See the chapter
> [Adding Resilience and Optimizing Performance][j_chapter7] for more
> information.

> **Note:** It is possible to have multiple **SubscriptionReceiver** 
> instances running, perhaps in multiple worker role instances. If 
> multiple **SubscriptionReceiver** instances can receive messages from 
> the same topic subscription, then the first one to call the **Receive** 
> method on the **SubscriptionClient** object will get and handle the 
> command. 

An alternative approach is to use a Windows Azure Service Bus queue in 
place of a topic for delivering command messages. Windows Azure Service 
Bus queues differ from topics in that they are designed to deliver 
messages to a single recipient instead of to multiple recipients through 
multiple subscriptions. The developers plan to evaluate this option in 
more detail with the intention of implementing this approach later in the 
project. 

The following code sample from the **SubscriptionReceiver** class shows 
how it receives a message from the topic subscription. 

```Cs
private SubscriptionClient client;

...

private void ReceiveMessages(CancellationToken cancellationToken)
{
    while (!cancellationToken.IsCancellationRequested)
    {
        BrokeredMessage message = null;

        try
        {
            message = this.receiveRetryPolicy.ExecuteAction<BrokeredMessage>(this.DoReceiveMessage);
        }
        catch (Exception e)
        {
            Trace.TraceError("An unrecoverable error occurred while trying to receive a new message:\r\n{0}", e);

            throw;
        }

        try
        {
            if (message == null)
            {
                Thread.Sleep(100);
                continue;
            }

            this.MessageReceived(this, new BrokeredMessageEventArgs(message));
        }
        finally
        {
            if (message != null)
            {
                message.Dispose();
            }
        }
    }
}

protected virtual BrokeredMessage DoReceiveMessage()
{
    return this.client.Receive(TimeSpan.FromSeconds(10));
}
```

> **JanaPersona:** This code sample shows how the system uses the
> [Transient Fault Handling Application Block][tfab] to retrieve 
> messages reliably from the topic.

The Windows Azure Service Bus **SubscriptionClient** class uses a 
peek/lock technique to retrieve a message from a subscription. In the 
code sample, the **Receive** method locks the message on the 
subscription. While the message is locked, other clients cannot see it. 
The **Receive** method then tries to process the message. 
If the client processes the message successfully, it calls the 
**Complete** method; this deletes the message from the subscription. 
Otherwise, if the client fails to process the message successfully, it 
calls the **Abandon** method; this releases the lock on the message and 
the same, or a different client can then receive it. If the 
client does not call either the **Complete** or **Abandon** methods 
within a fixed time, the lock on the message is released. 

> **Note:** The **MessageReceived** event passes a reference to the 
> **SubscriptionReceiver** instance so that the handler can call either 
> the **Complete** or **Abandon** methods when it processes the message.

The following code sample from the **MessageProcessor** class shows how 
to call the **Complete** and **Abandon** methods using
the **BrokeredMessage** instance passed as a parameter to the 
**MessageReceived** event. 

```Cs
private void OnMessageReceived(object sender, BrokeredMessageEventArgs args)
{
    var message = args.Message;

    object payload;
    using (var stream = message.GetBody<Stream>())
    using (var reader = new StreamReader(stream))
    {
        payload = this.serializer.Deserialize(reader);
    }

    try
    {
        ...

        ProcessMessage(payload);

        ...
    }
    catch (Exception e)
    {
        if (args.Message.DeliveryCount > MaxProcessingRetries)
        {
            Trace.TraceWarning("An error occurred while processing a new message and will be dead-lettered:\r\n{0}", e);
            message.SafeDeadLetter(e.Message, e.ToString());
        }
        else
        {
            Trace.TraceWarning("An error occurred while processing a new message and will be abandoned:\r\n{0}", e);
            message.SafeAbandon();
        }

        return;
    }

    Trace.TraceInformation("The message has been processed and will be completed.");
    message.SafeComplete();
}
``` 

> **Note:** This example uses an extension method to invoke the
> **Complete** and **Abandon** methods of the **BrokeredMessage**
> reliably using the [Transient Fault Handling Application Block][tfab].

### Why have separate CommandBus and EventBus classes?

Although at this early stage in the development of the Conference 
Management system the implementations of the **CommandBus** and 
**EventBus** classes are very similar and you may wonder why we have both, the team anticipates that they 
will diverge in the future. 

> **DeveloperPersona:** There may be differences in how we invoke 
> handlers and what context we capture for them: commands may want to 
> capture additional runtime state, whereas events typically don't need 
> to. Because of these potential future differences, I didn't want to 
> unify the implementations. I've been there before and ended up 
> splitting them when further requirements came in. 

### How scalable is this approach?

With this approach, you can run multiple instances of the 
**SubscriptionReceiver** class and the various handlers in different 
Windows Azure worker role instances, which enables you to scale out your 
solution. You can also have multiple instances of the **CommandBus**, 
**EventBus**, and **TopicSender** classes in different Windows Azure 
worker role instances. 

For information about scaling the Windows Azure Service Bus 
infrastructure, see [Best Practices for Performance Improvements Using 
Service Bus Brokered Messaging][sbperf] on MSDN. 

### How robust is this approach?

This approach uses the brokered messaging option of the Windows Azure 
Service Bus to provide asynchronous messaging. The Service Bus reliably 
stores messages until consumers connect and retrieve their messages. 

Also, the peek/lock approach to retrieving messages from a queue or 
topic subscription adds reliability in the scenario in which a message 
consumer fails while it is processing the message. If a consumer fails 
before it calls the **Complete** method, the message is still available 
for processing when the consumer restarts. 

### What is the granularity of a topic and a subscription?

The current implementation uses a single topic (**conference/commands**) 
for all commands within the system, and a single topic 
(**conference/events**) for all events within the system. There is a 
single subscription for each topic, and each subscription receives all 
of the messages published to the topic. It is the responsibility of the 
**CommandProcessor** and **EventProcessor** classes to deliver the 
messages to the correct handlers. 

In the future, the team will examine the options of using multiple topics &mdash; for example, using a separate command topic for each bounded context; and multiple subscriptions &mdash; such as one per event type. These alternatives may simplify the code and facilitate scaling of the application across multiple worker roles. 

> **JanaPersona:** There are no costs associated with having 
> multiple topics, subscriptions, or queues. Windows Azure Service Bus 
> usage is billed based on the number of messages sent and the amount of 
> data transferred out of a Windows Azure sub-region. 

### How are commands and events serialized?

The Contoso Conference Management System uses the [Json.NET][jsonnet] 
serializer. For details on how the application uses this serializer,
see [Technologies Used in the Reference Implementation][r_chapter9] in
the Reference Guide. 

> "You should consider whether you always need to use the Windows Azure
> Service Bus for commands. Commands are typically used within a bounded
> context and you may not need to send them across a process boundary
> (on the write-side you may not need additional tiers), in which case
> you could use an in memory queue to deliver your commands."
  
> &mdash; Greg Young, conversation with the patterns and practices team

# Impact on testing

Because this was the first bounded context the team tackled, one of the key concerns was how to approach testing given that the team wanted to adopt a test-driven development approach. The following conversation between two developers about how to do TDD when they are implementing the CQRS pattern without event sourcing summarizes their thoughts: 


> *Developer 1*: If we were using event sourcing, it would be easy to use 
> a TDD approach when we were creating our domain objects. The input to the 
> test would be a command (that perhaps originated in the UI), and we 
> could then test that the domain object fires the expected events. 
> However if we're not using event sourcing, we don't have any events: the 
> behavior of the domain object is to persist its changes in data store 
> through an ORM layer. 

> *Developer 2*: So why don't we raise events anyway? Just because we're 
> not using event sourcing doesn't mean that our domain objects can't 
> raise events. We can then design our tests in the usual way to check for 
> the correct events firing in response to a command. 

> *Developer 1*: Isn't that just making things more complicated than they 
> need to be? One of the motivations for using CQRS is to simplify things! 
> We now have domain objects that need to persist their state using an ORM 
> layer and raise events that report on what they have persisted just 
> so we can run our unit tests. 

> *Developer 2*: I see what you mean. 

> *Developer 1*: Perhaps we're getting stuck on how we're doing the 
> tests. Maybe instead of designing our tests based on the expected 
> *behavior* of the domain objects, we should think about testing the 
> *state* of the domain objects after they've processed a command. 

> *Developer 2*: That should be easy to do; after all, the domain objects 
> will have all of the data we want to check stored in properties so that 
> the ORM can persist the right information to the store. 

> *Developer 1*: So we really just need to think about a different style 
> of testing in this scenario. 

> *Developer 2*: There is another aspect of this we'll need to consider: 
> we might have a set of tests that we can use to test our domain objects, 
> and all of those tests might be passing. We might also have a set of 
> tests to verify that our ORM layer can save and retrieve objects 
> successfully. However, we will also have to test that our domain objects 
> function correctly when we run them against the ORM layer. It's possible 
> that a domain object performs the correct business logic, but can't 
> properly persist its state, perhaps because of a problem related to how 
> the ORM handles specific data types. 

For more information about the two approaches to testing discussed here, 
see Martin Fowler's article [Mocks Aren't Stubs][tddstyle] and
[Point/Counterpoint][point] by Steve Freeman, Nat Pryce, and Joshua
Kerievsky.

> **Note:** The tests included in the solution are written using
> xUnit.net.

The following code sample shows two examples of tests written using the 
behavioral approach discussed above. 

> **MarkusPersona:** These are the tests we started with, but we then
  replaced them with state-based tests.

```Cs
public SeatsAvailability given_available_seats()
{
	var sut = new SeatsAvailability(SeatTypeId);
	sut.AddSeats(10);
	return sut;
}
		
[TestMethod]
public void when_reserving_less_seats_than_total_then_succeeds()
{
	var sut = this.given_available_seats();
	sut.MakeReservation(Guid.NewGuid(), 4);
}

[TestMethod]
[ExpectedException(typeof(ArgumentOutOfRangeException))]
public void when_reserving_more_seats_than_total_then_fails()
{
	var sut = this.given_available_seats();
	sut.MakeReservation(Guid.NewGuid(), 11);
}
```

These two tests work together to verify the behavior of the 
**SeatsAvailability** aggregate. In the first test, the 
expected behavior is that the **MakeReservation** method succeeds and 
does not throw an exception. In the second test, the expected behavior 
is for the **MakeReservation** method to throw an exception because 
there are not enough free seats available to complete the reservation. 

It is difficult to test the behavior in any other way without the 
aggregate raising events. For example, if you tried to test the behavior 
by checking that the correct call is made to persist the aggregate to 
the data store, the test becomes coupled to the data store 
implementation (which is a smell); if you want to change the data store
implementation, you will need to change the tests on the aggregates in
the domain model. 

The following code sample shows an example of a test written using the 
state of the objects under test. This style of test is the one used
in the project.

```Cs
public class given_available_seats
{
	private static readonly Guid SeatTypeId = Guid.NewGuid();

	private SeatsAvailability sut;
	private IPersistenceProvider sutProvider;

	protected given_available_seats(IPersistenceProvider sutProvider)
	{
		this.sutProvider = sutProvider;
		this.sut = new SeatsAvailability(SeatTypeId);
		this.sut.AddSeats(10);

		this.sut = this.sutProvider.PersistReload(this.sut);
	}

	public given_available_seats()
		: this(new NoPersistenceProvider())
	{
	}

	[Fact]
	public void when_reserving_less_seats_than_total_then_seats_become_unavailable()
	{
		this.sut.MakeReservation(Guid.NewGuid(), 4);
		this.sut = this.sutProvider.PersistReload(this.sut);

		Assert.Equal(6, this.sut.RemainingSeats);
	}

	[Fact]
	public void when_reserving_more_seats_than_total_then_rejects()
	{
        var id = Guid.NewGuid();
        sut.MakeReservation(id, 11);

        Assert.Equal(1, sut.Events.Count());
        Assert.Equal(id, ((ReservationRejected)sut.Events.Single()).ReservationId);
	}
}
```

The two tests shown here test the state of the **SeatsAvailability** 
aggregate after invoking the **MakeReservation** method. The first test 
tests the scenario in which there are enough seats available. The second 
test tests the scenario in which there are not enough seats available. 
This second test can make use of the behavior of the **SeatsAvailability** 
aggregate because the aggregate does raise an event if it rejects a 
reservation. 

[j_chapter4]:     Journey_04_ExtendingEnhancing.markdown
[j_chapter5]:     Journey_05_PaymentsBC.markdown
[j_chapter6]:     Journey_06_V2Release.markdown
[j_chapter7]:     Journey_07_V3Release.markdown
[r_chapter3]:     Reference_03_ESIntroduction.markdown
[r_chapter4]:     Reference_04_DeepDive.markdown
[r_chapter6]:     Reference_06_Sagas.markdown
[r_chapter9]:     Reference_09_Technologies.markdown
[appendix1]:      Appendix1_Running.markdown

[tddstyle]:       http://martinfowler.com/articles/mocksArentStubs.html
[repourl]:        https://github.com/mspnp/cqrs-journey-code
[res-pat]:        http://www.rgoarchitects.com/nblog/2009/09/08/SOAPatternsReservations.aspx
[sbperf]:         http://msdn.microsoft.com/en-us/library/hh528527.aspx
[jsonnet]:        http://james.newtonking.com/pages/json-net.aspx
[sagapaper]:      http://www.amundsen.com/downloads/sagas.pdf
[tfab]:           http://msdn.microsoft.com/en-us/library/hh680934(PandP.50).aspx
[cqrsjourneysite]: http://cqrsjourney.github.com/
[memento]:        http://www.oodesign.com/memento-pattern.html
[downloadc]:      http://NEEDFWLINK
[prg]:            http://en.wikipedia.org/wiki/Post/Redirect/Get
[sbq]:            http://msdn.microsoft.com/en-us/library/windowsazure/hh767287.aspx
[point]:          http://doi.ieeecomputersociety.org/10.1109/MS.2007.84

[fig1]:           images/OrderMockup.png?raw=true
[fig2]:           images/Journey_03_Aggregates_01.png?raw=true
[fig3]:           images/Journey_03_Aggregates_02.png?raw=true
[fig4]:           images/Journey_03_Aggregates_03.png?raw=true
[fig5]:           images/Journey_03_Architecture_01.png?raw=true
[fig6]:           images/Journey_03_Architecture_02.png?raw=true
[fig7]:           images/Journey_03_Sequence_01.png?raw=true
[fig8]:           images/Journey_03_ServiceBus_01.png?raw=true
