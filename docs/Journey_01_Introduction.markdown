### This version of this chapter was part of our working repository during the project. The final version of this chapter is now available on MSDN at [http://aka.ms/cqrs](http://aka.ms/cqrs).

> # Chapter 1: Our Domain: The Contoso Conference Management System 

# 第1章: コントソ会議システム

> _The starting point: Where have we come from, what are we taking, and who is coming with us?_

_出発点: 我々はどこからきて、何を持ち、誰と共に行くのか？_

> "Without stones there is no arch," Marco Polo

> "石がなければアーチを作ることはできません" Marco Polo

> This chapter introduces a fictitious company named Contoso. It describes 
> Contoso's plans to launch the Contoso Conference Management System, a 
> new online service that will enable other companies or individuals to 
> organize and manage their own conferences and events. This chapter 
> describes, at a high-level, some of the functional and non-functional 
> requirements of the new system, and why Contoso wants to implement parts 
> of it using the Command Query Responsibility Segregation (CQRS) pattern and event sourcing (ES). As with any 
> company considering this process, there are many issues to consider and 
> challenges to be met, particularly because this is the first time 
> Contoso has used both the CQRS pattern and event sourcing. The chapters 
> that follow show, step by step, how Contoso designed and 
> built its conference management application. 

本章では、コントソ社という架空の企業と、コントソ社が計画しているコントソ会議システムについて説明します。
コントソ会議システムは、企業や個人が自分のカンファレンスやイベントを企画、管理するための新しいオンラインサービスです。
本章では、新システムの機能要求・非機能要求と、その一部を CQRS（コマンド・クエリ責務分離）パターンとES（Event Sourcing）を用いて実装する理由について、大まかに説明します。
こういったプロセスを検討している様々な企業と同様に、このプロジェクトには検討すべき課題が数多くあり、
特にコントソ社はCQRSパターンとイベントソーシング使用するのが両方とも初めてなので課題が沢山ありした。
この後の章では、コントソ社が会議管理アプリケーションを設計・構築した方法を順を追って説明します。

> This chapter also introduces a panel of fictional experts to comment on the development efforts. 

また、本章では、開発の取り組みについてコメントをした架空の専門家パネルを紹介します。

> # The Contoso Corporation 

# コントソ社

> Contoso is a startup ISV company of approximately 20 employees that 
> specializes in developing solutions using Microsoft technologies. 
> The developers at Contoso are knowledgeable about various Microsoft 
> products and technologies, including the .NET Framework, ASP.NET MVC, 
> and Windows Azure. Some of the developers have previous experience
> using the domain-driven design (DDD) approach, but none of them have 
> used the CQRS pattern previously. 

コントソ社は、従業員約20名のスタートアップ独立系ソフトウェアベンダで、
マイクロソフトの技術を使ったソリューションの開発を専門としています。
コンソト社の開発者は、.NET Framework、ASP.NET MVC、Windows Azureなど、 さまざまなMicrosoftの製品や技術に精通しています。
開発者の中には、ドメイン駆動設計（DDD）の経験者はいますが、CQRSの経験者はいません。

> The Conference Management System application is one of the first 
> innovative online services that Contoso wants to take to market. As a 
> startup, Contoso wants to develop and launch these services with a 
> minimal investment in hardware and IT personnel. Contoso wants to be 
> quick to market in order to start growing market share, and cannot 
> afford the time to implement all of the planned functionality in the 
> first releases. Therefore, it is important that the architecture it 
> adopts can easily accommodate changes and enhancements with minimal 
> impact on existing users of the system. Contoso has chosen to deploy the 
> application on Windows Azure in order to take advantage of its ability 
> to scale applications as demand grows. 

会議システムは、コントソ社がこれから売り出そうとしている最初の革新的なオンラインサービスの一つです。
新興企業であるコントソ社は、ハードウェアやIT人材への投資を最小限に抑えながら、このサービスの開発・提供を行いたいと考えています。
コントソ社はいち早く製品を市場に投入することでマーケットシェアを拡大したいと考えているので、 
最初のリリースで予定されているすべての機能を実装する時間的余裕はありません。
したがって、最初のリリース後に既存のシステムユーザーへの影響を最小限に抑えつつ、変更や機能拡張を容易に行えるアーキテクチャを採用することが重要です。
コントソ社は、需要の増加に応じてアプリケーションをスケールできるように、Windows Azure上にアプリケーションをデプロイすることにしました。

> # Who is coming with us on the journey? 

# 旅の供

> As mentioned earlier, this guide and the accompanying RI describe a CQRS 
> journey. A panel of experts will comment on our development efforts as we go. 
> This panel includes a CQRS expert, a software architect, a developer, a 
> domain expert, an IT Pro, and a business manager. They will all comment 
> from their own perspectives. 

先ほども述べたように、このガイドと付属の参照実装は、CQRSを巡る旅の物語です。
物語の中で専門家の講師が、私たちの開発の取り組みについてコメントしてくれます。
講師たちは、CQRSエキスパート、ソフトウェアアーキテクト、開発者、ドメインエキスパート、ITプロ、ビジネスマネージャーといった人々で、それぞれが独自の視点からコメントします。

<table border="1">
<tr>
  <td>
    <img src="images/PersonaGary.png?raw=true" />
  </td>
<td>
<blockquote>
Gary is a CQRS expert. He ensures that a CQRS-based solution will work 
for a company and will provide tangible benefits. He is a cautious 
person, for good reason.<br/>

<i>"Defining the CQRS pattern is easy. Realizing the benefits that
implementing the CQRS pattern can offer is not always so
straightforward."</i>
</blockquote>
GaryはCQRSのエキスパートで、CQRSに基づいた企業にとって有効なソリューションを提案し、具体的な利益をもたらすことを保証します。彼は訳があって慎重な人です。<br/>

<i>"CQRSパターンの定義そのものは簡単です。しかし、CQRSパターンを導入することで得られるメリットを享受するのは、必ずしもそれほど簡単ではありません。"</i>

</td>
</tr>

<tr>
  <td>
    <img src="images/PersonaJana.png?raw=true" />
  </td>
<td>
<blockquote>
Jana is a software architect. She plans the overall structure of an 
application. Her perspective is both practical and strategic. In other 
words, she considers not only what technical approaches are needed 
today, but also what direction a company needs to consider for the 
future. Jana has worked on projects that used the Domain-Driven Design 
approach.<br/>

<i>"It's not easy to balance the needs of the company, the users, the IT
organization, the developers, and the technical platforms we rely on."</i>
</blockquote>
Janaはソフトウェアアーキテクトで、アプリケーションの全体的な構造を設計します。
彼女の視点は実用的かつ戦略的で、今必要な技術的なアプローチだけでなく、企業が向かうべき将来的な方向性も考慮しています。
Janaは、ドメイン駆動設計を用いたプロジェクトに参加したことがあります。

<i>企業、ユーザー、IT組織、開発者、依存する技術プラットフォームのニーズのバランスを取るのは簡単ではありません。</i>
</td>
</tr>

<tr>
  <td>
    <img src="images/PersonaMarkus.png?raw=true" />
  </td>
<td>
<blockquote>
Markus is a software developer who is new to the CQRS pattern. He is 
analytical, detail-oriented, and methodical. He's focused on the task at 
hand, which is building a great application. He knows that he's the 
person who's ultimately responsible for the code.<br/>

<i>"I don't care what architecture you want to use for the application; I'll make it work."</i>
</blockquote>
Markusはソフトウェア開発者で、CQRSパターンの経験は浅いです。
彼は分析的で、細部に目が届き、几帳面な性格です。
彼は、優れたアプリケーションを構築するという、目の前のタスクに集中しています。
彼は、自分がコードの最終責任者であると自負しています。

<i>どんなアーキテクチャであっても、ちゃんと動くようにして見せます。</i>
</td>
</tr>

<tr>
  <td>
    <img src="images/PersonaCarlos.png?raw=true" />
  </td>
<td>
<blockquote>
Carlos is the domain expert. He understands all the ins and outs of 
conference management. He has worked in a number of organizations that 
help people run conferences. He has also worked in a number of 
different roles: sales and marketing, conference management, and 
consultant.<br/>

<i>"I want to make sure that the team understands how this business
works so that we can deliver a world-class online conference 
management system."</i>
</blockquote>
Carlosはドメインエキスパートで会議運営の裏も表も知り尽くしています。
彼は、いくつかのカンファレンスの運営を支援する組織で働いた経験があります。
また、営業やマーケティング、会議運営、コンサルタントなど、さまざまな役割を担ってきました。

<i>このビジネスの仕組みをチームに理解してもらい、世界に通用するオンライン会議管理システムを提供することが私の仕事です。</i>
</td>
</tr>

<tr>
  <td>
    <img src="images/PersonaPoe.png?raw=true" />
  </td>
<td>
<blockquote>
Poe is an IT professional who's an expert in deploying and running 
applications in the cloud. Poe has a keen interest in practical 
solutions; after all, he's the one who gets paged at 3:00 AM when 
there's a problem.<br/>

<i>"Running complex applications in the cloud involves challenges that
are different than the challenges in managing  on-premises applications.
I want to make sure our new conference management system meets our
published service-level agreements (SLA)."</i>
</blockquote>
Poeは、クラウド上でアプリケーションをデプロイ・実行することに熟達したITのプロです。
問題が発生すると午前3時に呼び出されることもあるので、実践的なソリューションに強い関心を持っています。

<i>クラウド上での複雑なアプリケーション運用には、オンプレミスなアプリケーションの管理とは違った課題があります。
新しい会議管理システムが、宣言したサービスレベルアグリーメント（SLA）を確実に満たせるようにしたいと思います。</i>
</td>
</tr>

<tr>
  <td>
    <img src="images/PersonaBeth.png?raw=true" />
  </td>
<td>
<blockquote>
Beth is a business manager. She helps companies to plan how their 
business will develop. She understands the market that the company 
operates in, the resources that the company has available, and the goals 
of the company. She has both a strategic view and an interest in the 
day-to-day operations of the company.<br/>

<i>"Organizations face many conflicting demands on their resources. I 
want to make sure that our company balances those demands and adopts a 
business plan that will make us successful in the medium and long 
term."</i>
</blockquote>
Bethはビジネスマネージャーで、企業のビジネス展開の計画を手助けします。
彼女は、企業が活動する市場、企業が利用できるリソース、企業の目標を理解していて、
戦略的な視点と企業の日常業務への関心の両方を持っています。

<i>組織の資源は限られている一方で相反する要求は数多くあります。
私は、当社がこれらの要求のバランスをとり、中長期的に成功するようなビジネスプランを採用するようにしたいと考えています。</i>
</td>

<i>
</tr>
</table>

If you have a particular area of interest, look for notes provided by 
the specialists whose interests align with yours. 

# The Contoso Conference Management System 

This section describes the Contoso Conference Management System as the  
team envisaged it at the start of the journey. The team has not 
used the CQRS pattern before; therefore, the system that is delivered at 
the end of our journey may not match this description exactly because: 

* What we learn as we go may impact what we ultimately deliver.
* Because this is a learning journey, it is more difficult to estimate
  what we can achieve in the available time.

## Overview of the system

Contoso plans to build an online conference management system that will 
enable its customers to plan and manage conferences that are held 
at a physical location. The system will enable Contoso's customers to: 

* Manage the sale of different seat types for the conference.
* Create a conference and define characteristics of that conference.

The Contoso Conference Management System will be a multi-tenant, 
cloud-hosted application. Business Customers will need to register with 
the system before they can create and manage their conferences. 

### Selling seats for a conference

The business customer defines the number of seats available for the 
conference. The business customer may also specify events at a 
conference such as workshops, receptions, and premium sessions for which 
attendees must have a separate ticket. The business customer also 
defines how many seats are available for these events. 

The system manages the sale of seats to ensure that the conference and 
sub-events are not oversubscribed. This part of the system will also 
operate wait-lists so that if other Attendees cancel, their seats 
can be reallocated. 

The system will require that the names of the Attendees be associated 
with the purchased seats so that an on-site system can print badges for
the Attendees when they arrive at the conference. 

### Creating a conference

A Business Customer can create new conferences and manage information 
about the conference such as its name, description, and dates. The 
Business Customer can also make a conference visible on the Contoso 
Conference Management System website by publishing it, or hide it by 
unpublishing it. 

Additionally, the Business Customer defines the seat types and available 
quantity of each seat type for the conference. 

Contoso also plans to enable the Business Customer to specify the 
following characteristics of a conference: 

* Whether the paper submission process will require reviewers.
* What the fee structure for paying Contoso will be.
* Who key personnel, such as the program chair and the event planner,
  will be.

## Nonfunctional requirements

Contoso has two major nonfunctional requirements for its conference
management system-scalability and flexibility-and it hopes that the CQRS
pattern will help it meet them. 

### Scalability

The Conference Management System will be hosted in the cloud; one of 
the reasons Contoso chose a cloud platform was its scalability and 
potential for elastic scalability. 

Although cloud platforms such as Windows Azure enable you to scale 
applications by adding (or removing) role instances, you must still 
design your application to be scalable. By splitting responsibility for 
the application's read and write operations into separate objects, the 
CQRS pattern allows Contoso to split those operations into separate 
Windows Azure roles that can scale independently of each other. This 
recognizes the fact that for many applications, the number of read 
operations vastly exceeds the number of write operations. This gives 
Contoso the opportunity to scale the Conference Management System more 
efficiently, and make better use of the Windows Azure role instances 
it uses. 

### Flexibility

The market that the Contoso Conference Management System operates in is 
very competitive, and very fast moving. In order to compete, Contoso 
must be able to quickly and cost effectively adapt the Conference 
Management System to changes in the market. This requirement for 
flexibility breaks down into a number of related aspects: 

* Contoso must be able to evolve the system to meet new requirements 
  and to respond to changes in the market. 
  
> **BethPersona:** Contoso plans to compete by being quick to respond to
> changes in the market and to changing customer requirements. Contoso
> must be able to evolve the system quickly and painlessly.

* The system must be able to run multiple versions of its software 
  simultaneously in order to support customers who are in the middle of 
  a conference and who do not wish to upgrade to a new version 
  immediately. Other customers may wish to migrate their existing 
  conference data to a new version of the software as it becomes 
  available.
  
> **PoePersona:** This is a big challenge: keeping the system running
> for all our customers while we perform upgrades with no down time.

* Contoso intends the software to last for at least five years. It 
  must be able to accommodate significant changes over that period. 

* Contoso does not want the complexity of some parts of the system to 
  become a barrier to change. 

* Contoso would like to be able to use different developers for 
  different elements of the system, using cheaper developers for simpler 
  tasks and restricting its use of more expensive and experienced 
  developers to the more critical aspects of the system. 

> **GaryPersona:** There is some debate in the CQRS community 
> about whether, in practice, you can use different development teams 
> for different parts of the CQRS pattern implementation. 

# Beginning the journey

The next chapter is the start of our CQRS journey. It provides more 
information about the Contoso Conference Management System and describes 
some of the high-level parts of the system. Subsequent chapters describe 
the stages of the journey as Contoso implements the Conference Management 
System. 

[personagary]:    images/PersonaGary.png?raw=true
[personajana]:    images/PersonaJana.png?raw=true
[personamarkus]:  images/PersonaMarkus.png?raw=true
[personacarlos]:  images/PersonaCarlos.png?raw=true
[personapoe]:     images/PersonaPoe.png?raw=true
[personabeth]:    images/PersonaBeth.png?raw=true