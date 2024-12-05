### This version of this chapter was part of our working repository during the project. The final version of this chapter is now available on MSDN at [http://aka.ms/cqrs](http://aka.ms/cqrs).

> # Reference 6: A Saga on Sagas (Chapter Title)

# 参照 6: サーガ・パターンにおけるサーガ (チャプターのタイトル)

> **Process Managers, Coordinating Workflows, and Sagas**

**プロセスマネージャ、ワークフローの調整、およびサーガ**

> # Clarifying the terminology

# 用語の明確化

> The term **Saga** is commonly used in discussions of CQRS to refer to a 
> piece of code that coordinates and routes messages between bounded 
> contexts and aggregates. However, for the purposes of this guidance we 
> prefer to use the term **Process Manager** to 
> refer to this type of code artefact. There are two reasons for this: 
> 
> 1. There is a well-known, pre-existing definition of the term **Saga**
>    that has a different meaning from the one generally understood in
>    relation to CQRS.
> 2. The term **Process Manager** is a better description of the
>    role performed by this type of code artefact.

CQRSに関する議論の中で、境界づけられたコンテキスト間や集約間のメッセージを調整しルーティングするコードの部品を一般的に **サーガ** と呼ぶことがあります。
しかし、このガイドにおいて、この種のコードには **プロセスマネージャ**という用語を使用することにしました。これには2つの理由があります。
   
>> **Make this a sidebar**
>> Although the term Saga is often used in the context of the CQRS
>> pattern, it has a pre-existing definition. We have chosen to use the
>> term process manager in this guidance to avoid confusion with this
>> pre-existing definition. 

> **ここはサイドバーにする**
> サーガという用語はCQRSパターンの文脈でよく使われますが、
> この用語にはもう決まっている明確な意味があります。このガイダンスでは、この既に決まっている意味との混同を避けるために、
> プロセスマネージャという用語を使用することにしました。
>
>> The term saga, in relation to distributed systems, was originally
>> defined in the paper [Sagas](sagapaper) by Hector Garcia-Molina and
>> Kenneth Salem. This paper proposes a mechanism that it calls a saga as
>> an alternative to using a distributed transaction for managing a
>> long-running business process. The paper recognizes that business
>> processes are often comprised of multiple steps, each one of which
>> involves a transaction, and that overall consistency can be achieved
>> by grouping these individual transactions into a distributed
>> transaction. However, in long-running business processes, using
>> distributed transactions can impact on the performance and concurrency
>> of the system because of the locks that must be held for the duration
>> of the distributed transaction. 
>
> サーガという用語は、Hector Garcia-MolinaとKenneth Salemによる分散システムに関する論文 [Sagas](sagapaper) で定義されたものに由来します。
> この論文では、長期間にわたるビジネスプロセスを管理するための分散トランザクションの代わりとして、サーガというメカニズムを提案しています。 
> この論文は、ビジネスプロセスがしばしばトランザクションを伴う複数のステップで構成されることを示し、
> これらの個々のトランザクションを分散トランザクションにグループ化することで全体的な一貫性を実現できるという主張をしています。
> しかし、長期間にわたるビジネスプロセスでは、分散トランザクションを使用することが、
> 分散トランザクションの期間中に保持するロックによってシステムのパフォーマンスと並行性に影響を与える可能性があるとしています。
> 
>> The saga concept removes the need for a distributed transaction by
>> ensuring that the transaction at each step of the business process has
>> a defined compensating transaction. In this way, if the business
>> process encounters an error condition and is unable to continue, it
>> can execute the compensating transactions for the steps that have
>> already completed. This undoes the work completed so far in the
>> business process and maintains the consistency of the system. 
> 
> サーガの概念を用いると、ビジネスプロセスの各ステップでトランザクションが定義された補償トランザクションを持つことによって、
> 分散トランザクションの必要性を取り除きます。この方法により、ビジネスプロセスがエラー条件に遭遇して続行できない場合、
> 既に完了したステップに対する補償トランザクションを実行します。
> これにより、これまでに完了したビジネスプロセスの作業を元に戻し、システムの一貫性を維持します。

> Although we have chosen to use the term process manager, Sagas (as 
> defined in the [paper][sagapaper] by Hector Garcia-Molina and Kenneth 
> Salem) may still have a part to play in a system that implements the 
> CQRS pattern in some of its bounded contexts. Typically, you would 
> expect to see a process manager routing messages between aggregates 
> within a bounded context, and you would expect to see a saga managing a 
> long-running business process that spans multiple bounded contexts. 

このガイドでは、プロセスマネージャという用語を使用することにしましたが、CQRSパターンを実装するシステムにおいて、
いくつかの境界づけられたコンテキストでサーガ（Hector Garcia-MolinaとKenneth Salemによる[論文][sagapaper]で定義されたもの）が依然として重要な役割を果たす可能性があります。
例えば、プロセスマネージャが境界づけられたコンテキスト内の集約間のメッセージをルーティングする役割を担い、
サーガが複数の境界づけられたコンテキストにまたがる長期間にわたるビジネスプロセスを管理する役割を担う、という形です。

> The following section describes what we mean by the term **Process 
> Manager**. This is the working definition we used during our CQRS 
> journey project. 

次のセクションでは、**プロセスマネージャ**という用語の意味について説明します。
これは、CQRSを巡るたびのプロジェクトで実際に使用した定義です。

>> **Note:** For a time the team developing the Reference Implementation
>> used the term **Coordinating Workflow** before settling on the term
>> **Process Manager**. This pattern is described in the book "Enterprise
>> Integration Patterns" by Gregor Hohpe and Bobby Woolf.

> **注:** 一時期、リファレンス実装を開発していたチームは、**プロセスマネージャ**という用語を確定する前に、
> **ワークフローの調整**という用語を使用していました。
> このパターンは、Gregor HohpeとBobby Woolfによる書籍「Enterprise Integration Patterns」に出てきます。

> # Process Manager

# プロセスマネージャ

> This section outlines our definition of the term **Process Manager**. 
> Before describing the **Process Manager** there is a 
> brief recap of how CQRS typically uses messages to communicate between 
> aggregates and bounded contexts. 

このセクションでは、**プロセスマネージャ** という用語の定義について概説します。
**プロセスマネージャ** を説明する前に、CQRSが通常どのようにメッセージを使って集約間や境界づけられたコンテキスト間で通信するかについて、簡単に振り返ります。

> ## Messages and CQRS

## メッセージとCQRS

> When you implement the CQRS pattern, you typically think about two types 
> of message to exchange information within your system: commands and 
> events. 

CQRSパターンによる実装では、通常、システム内でコマンドとイベントという2種類のメッセージを使って情報をやり取りします。

> Commands are imperatives; they are requests for the system to 
> perform a task or action. For example, "book two places on conference X" 
> or "allocate speaker Y to room Z." Commands are usually processed just 
> once, by a single recipient.

コマンドはシステムにタスクやアクションを実行するように要求する命令です。
例えば、「会議Xの2つの席を予約せよ」や「スピーカーYを部屋Zに割り当てよ」といった形です。
コマンドは通常、1つの受信者が1度だけ処理します。

> Events are notifications; they inform interested parties that something 
> has happened. For example, "the payment was rejected" or "seat type X 
> was created." Notice how they use the past tense. Events are published 
> and may have multiple subscribers. 

関係者に何かが起こったことを知らせる通知です。
例えば、「支払いが拒否された」や「座席タイプXが作成された」といった形です。
過去形を使って通知していることに注意してください。
イベントが発行されると、複数の購読者がそれを購読する可能性があります。

> Typically, commands are sent within a bounded context. Events may have 
> subscribers in the same bounded context as where they are published, or 
> in other bounded contexts. 

通常、コマンドは境界づけられたコンテキスト内で送信されます。
イベントは、発行された境界づけられたコンテキスト内や他の境界づけられたコンテキスト内で購読者を持つことがあります。

> The chapter [A CQRS and ES Deep Dive][r_chapter4] in this Reference Guide 
> describes the differences between these two message types in detail. 

このリファレンスガイドの[CQRSとESについて深掘りする][r_chapter4]の章では、これら2種類のメッセージの違いについて詳しく説明しています。

> ## What is a Process Manager?

## プロセスマネージャとは何か？

> In a complex system that you have modelled using aggregates and bounded 
> contexts, there may be some business processes that involve multiple 
> aggregates, or multiple aggregates in multiple bounded contexts. In 
> these business processes multiple messages of different types are 
> exchanged by the participating aggregates. For example, in a conference 
> management system, the business process of purchasing seats at a 
> conference might involve an order aggregate, a reservation aggregate, 
> and a payment aggregate. They must all cooperate to enable a customer to 
> complete a purchase. 

集約と境界づけられたコンテキストを使ってモデル化した複雑なシステムでは、複数の集約や異なる境界づけられたコンテキストの集約にまたがる複数のビジネスプロセスがあるかもしれません。
このようなビジネスプロセスでは、参加する集約間で異なる種類の複数のメッセージを交換します。
例えば、会議管理システムでは、会議の席を購入するビジネスプロセスでは、注文集約、予約集約、支払い集約が関わるでしょう。
これらがすべて、顧客が購入を完了できるように協力する必要があります。

> Figure 1 shows a simplified view of the messages that these aggregates 
> might exchange to complete an order. The numbers identify the message 
> sequence. 

図1に、これらの集約が注文を完了するために交換するメッセージの様子を簡単に示します。
数字はメッセージの順序を示しています。

> **Note:** This does not illustrate how the Reference Implementation
> processes orders.

<!--
![Figure 1][fig1]
-->
![図1][fig1]

> **Order processing without using a Process Manager**

**プロセスマネージャを使わない注文処理**

> In the example shown in Figure 1, each aggregate sends the appropriate 
> command to the aggregate that performs the next step in the process. The 
> **Order** aggregate first sends a **MakeReservation** command to the 
> **Reservation** aggregate to reserve the seats requested by the 
> customer. After the seats have been reserved, the **Reservation** 
> aggregate raises a **SeatsReserved** event to notify the **Order** 
> aggregate, and the **Order** aggregate sends a **MakePayment** command 
> to the **Payment** aggregate. If the payment is successful, the 
> **Order** aggregate raises an **OrderConfirmed** event to notify the 
> **Reservation** aggregate that it can confirm the seat reservation, and 
> the customer that the order is now complete. 

図1に示している例では、各集約は、プロセスの次のステップを実行する集約に必要なコマンドを送信します。
**注文(Order)** 集約は最初に、顧客がリクエストした席を予約するために **予約(Reservation)** 集約に **MakeReservation** コマンドを送信します。
席が確保できたら、**予約(Reservation)** 集約は **SeatsReserved** イベントを発生させて **注文(Order)** 集約に通知し、
**注文(Order)** 集約は **MakePayment** コマンドを **支払い(Payment)** 集約に送信します。
支払いが成功したら、**注文(Order)** 集約は **OrderConfirmed** イベントを発生させて、**予約(Reservation)** 集約が座席予約を確定できるようにし、
顧客に注文が完了したことを通知します。

![Figure 2][fig2]

> **Order processing with a Process Manager**

**プロセスマネージャを使った注文処理**

> The example shown in Figure 2 illustrates the same business process as 
> that shown in Figure 1, but this time using a Process Manager. 
> Now, instead of each aggregate sending messages directly to other 
> aggregates, the messages are mediated by the Process Manager. 

図2に示している例は、図1に示されているものと同じビジネスプロセスを示していますが、今回はプロセスマネージャを使っています。
今回は、各集約が直接他の集約にメッセージを送信するのではなく、プロセスマネージャがメッセージの送信を仲介します。

> This appears to complicate the process: there is an additional object 
> (the Process Manager) and a few more messages. However, there are 
> benefits to this approach. 

このプロセスは追加のオブジェクト(プロセスマネージャ)を必要とし、一見複雑ですが、このアプローチにはいくつかの利点もあります。

> Firstly, the aggregates no longer need to know what is the next step in 
> the process. Originally, the **Order** aggregate needed to know that 
> after making a reservation it should try to make a payment by sending a 
> message to the **Payment** aggregate. Now, it simply needs to report 
> that an order has been created. 

まず、集約がプロセスの次のステップが何であるかを知る必要がなくなります。
元々、**注文(Order)** 集約は、予約を行った後に **支払い(Payment)** 集約にメッセージを送信して支払いを試みるべきであることを知る必要がありました。
今回は、単に注文が作成されたことを報告するだけで済みます。

> Secondly, the definition of the message flow is now located in a single 
> place, the Process Manager, rather than being scattered throughout 
> the aggregates. 

次に、メッセージのフローの定義が、集約全体に分散しているのではなく、プロセスマネージャの1か所に集中しているため、
メッセージのフローの定義を1か所にまとめることができます。

> In a simple business process such as the one shown in Figure 1 and 
> Figure 2, these benefits are marginal. However, if you have a business 
> process that involves six aggregates and tens of messages, the benefits 
> become more apparent. This is espcially true if this is a volatile part 
> of the system where there are frequent changes to the business process: 
> in this scenario, the changes are likely to be localized to a limited 
> numbe of objects. 

図1と図2に示されているような単純なビジネスプロセスでは、これらの利点は限定的です。
しかし、6つの集約と数十のメッセージを含むビジネスプロセスがある場合、これは明確な利点になります。
特に、ビジネスプロセスの変更が頻繁に行われるシステムの不安定な部分である場合、
変更を限られた数のオブジェクトに局所化できる可能性が高いため、この利点が特に真価を発揮します。

> In Figure 3, to illustrate this point, we introduce wait-listing to the 
> process. If some of the seats requested by the customer cannot be 
> reserved, the system adds these seat requests to a wait-list. To make 
> this change, we modify the **Reservation** aggregate to raise a 
> **SeatsNotReserved** event to report how many seats could not be 
> reserved in addition to the **SeatsReserved** event that reports how 
> many seats could be reserved. The Process Manager can then send a 
> command to the **WaitList** aggregate to wait-list the unfulfilled part 
> of the request. 

この点を説明するために、図3では、プロセスに待ちリストを導入しています。
顧客がリクエストした席の一部を予約できない場合、システムはこれらの席リクエストを待ちリストに追加します。
この変更を行うために、**予約(Reservation)** 集約を変更して、**SeatsReserved** イベントに加えて、
予約できなかった席の数を報告する **SeatsNotReserved** イベントを発生させるようにします。
その後、プロセスマネージャは、リクエストの未処理部分を待ちリストに登録するために **WaitList** 集約にコマンドを送信できます。

<!--
![Figure 3][fig3]
-->
![図3][fig3]

> It's important to note that the Process Manager does not perform 
> any business logic. It only routes messages, and in some cases 
> translates between message types. For example, when it receives a 
> **SeatsNotReserved** event, it sends an **AddToWaitList** command. 

プロセスマネージャがビジネスロジックを実行しないことに注意することが重要です。
プロセスマネージャは、メッセージのルーティングのみを行い、必要に応じてメッセージの種類を変換します。
例えば、**SeatsNotReserved** イベントを受信した場合、**AddToWaitList** コマンドを送信します。

> ## When should I use a Process Manager?

## どのような場合にプロセスマネージャを使うのか

> Process Manager route commands and events between aggregate 
> instances, however they don't implement any business logic. There are 
> two key reasons to use a Process Manager: 
> 
> * When your bounded context uses a large number of events and commands
>   that would be difficult to manage as a collection point-to-point
>   interactions between aggregates.
> * When you want to make it easier to modify message routing in the
>   bounded context. A Process Manager gives a single place where
>   the routing is defined.

プロセスマネージャは、集約インスタンス間でコマンドとイベントをルーティングしますが、ビジネスロジックは実装しません。
プロセスマネージャを使用する理由は以下の2つです。

* 境界づけられたコンテキストが多数のイベントとコマンドを使用しており、集約間のポイントツーポイントの相互作用を管理するのが難しい場合。
* 境界づけられたコンテキスト内のメッセージルーティングを変更しやすくしたい場合。プロセスマネージャを使うことで、ルーティングを定義する場所を1箇所にまとめることができます。

> ## When should I not use a Process Manager?

## プロセスマネージャを使うべきではない場合

> The following list identifies reasons not to use a Process Manager:
> 
> * You should not use a Process Manager if your bounded context
>   contains a small number of aggregate types that use a limited number
>   of messages. 
> * You should not use a Process Manager to implement any business
>   logic in your domain. Business logic belongs in the aggregate types.

以下においてはプロセスマネージャを使うべきではないでしょう。

* 境界づけられたコンテキストが、限られた数のメッセージを使用する少数の種類の集約しかない場合
* ドメイン内のビジネスロジックを実装するためにプロセスマネージャを使用すべきではありません。ビジネスロジックは集約に属します。
 

> ## Sagas and CQRS

## サーガとCQRS

> Although we have chosen to use the term process manager as defined 
> earlier in this chapter, Sagas (as defined in the paper by Hector 
> Garcia-Molina and Kenneth Salem) may still have a part to play in a 
> system that implements the CQRS pattern in some of its bounded contexts. 
> Typically, you would expect to see a process manager routing 
> messages between aggregates within a bounded context, and you would 
> expect to see a saga managing a long-running business process that spans 
> multiple bounded contexts. 

この章の前半で定義したように、プロセスマネージャという用語を使用することにしましたが、
サーガ（Hector Garcia-MolinaとKenneth Salemによる論文で定義されたもの）は、
CQRSパターンを実装するシステムの一部の境界づけられたコンテキストで依然として重要な役割を果たすことがあります。
通常、境界づけられたコンテキスト内の集約間のメッセージをルーティング役割はプロセスマネージャが担い、
複数の境界づけられたコンテキストにまたがる長期間にわたるビジネスプロセスを管理する役割はサーガが担うことが期待されます。


[r_chapter4]:     Reference_04_DeepDive.markdown
[sagapaper]:      http://www.amundsen.com/downloads/sagas.pdf

[fig1]:           images/Reference_06_Naive.png?raw=true
[fig2]:           images/Reference_06_Workflow.png?raw=true
[fig3]:           images/Reference_06_WorkflowExtended.png?raw=true