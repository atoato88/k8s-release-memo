本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

---

# Kubernetes Blog より

Original Post: https://kubernetes.io/blog/2021/08/04/kubernetes-1-22-release-announcement/

**Authors:** [Kubernetes 1.22 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.22/release-team.md)

We’re pleased to announce the release of Kubernetes 1.22, the second release of 2021!

2021年の2回目のリリースであるKubernetes 1.22のリリースをアナウンスできることを嬉しく思います。

This release consists of 53 enhancements: 13 enhancements have graduated to stable, 24 enhancements are moving to beta, and 16 enhancements are entering alpha. Also, three features have been deprecated.

このリリースでは53個のEnhancementがありました。13個がStableとなり、24個がBetaとなり、16個がAlphaとなりました。また、3つがDeprecatedとなりました。

In April of this year, the Kubernetes release cadence was officially changed from four to three releases yearly. This is the first longer-cycle release related to that change. As the Kubernetes project matures, the number of enhancements per cycle grows. This means more work, from version to version, for the contributor community and Release Engineering team, and it can put pressure on the end-user community to stay up-to-date with releases containing increasingly more features.

今年の4月にKuberenetesのリリースに要する期間は公式に年間4回から3回に変更されました。これは変更後はじめての長期サイクルのリリースとなります。Kubernetesプロジェクトが成熟するに従って、サイクルごとのEnhancementの数は増えています。これはバージョンごとにコントリビュータコミュニティとリリースエンジニアリングチームにとって作業が増えるということです。また、エンドユーザコミュニティに対して益々多くの機能を備えたリリースに追従することを推し進めるでしょう。

Changing the release cadence from four to three releases yearly balances many aspects of the project, both in how contributions and releases are managed, and also in the community's ability to plan for upgrades and stay up to date.

リリースに要する期間の年間4回から3回への変更はコントリビューションとリリースの管理と、アップグレードの計画により最新版を保つ能力の両方に置いて、プロジェクトの多くの側面でバランスが取れます。

You can read more in the official blog post [Kubernetes Release Cadence Change: Here’s What You Need To Know](https://kubernetes.io/blog/2021/07/20/new-kubernetes-release-cadence/).

公式のブログ [Kubernetes Release Cadence Change: Here’s What You Need To Know](https://kubernetes.io/blog/2021/07/20/new-kubernetes-release-cadence/) で詳細を読むことが出来ます。


## Major Themes

### Server-side Apply graduates to GA

[Server-side Apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/) is a new field ownership and object merge algorithm running on the Kubernetes API server. Server-side Apply helps users and controllers manage their resources via declarative configurations. It allows them to create and/or modify their objects declaratively, simply by sending their fully specified intent. After being in beta for a couple releases, Server-side Apply is now generally available.

[Server-side Apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/)は新しいフィールドの所有権であり、Kubernetes API Serverで動作するオブジェクトのマージアルゴリズムです。Server-side Applyはユーザとコントローラが宣言的な構成でリソースを管理することを助けます。これによりオブジェクトを宣言的に作成、更新することが可能になり、シンプルに全てを指定した意図を送信するだけでよくなります。いくつかのリリースでベータだった後に、Server-side ApplyはGA(Generally Available)となります。

### External credential providers now stable

Support for Kubernetes client [credential plugins](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins) has been in beta since 1.11, and with the release of Kubernetes 1.22 now graduates to stable. The GA feature set includes improved support for plugins that provide interactive login flows, as well as a number of bug fixes. Aspiring plugin authors can look at [sample-exec-plugin](https://github.com/ankeesler/sample-exec-plugin) to get started.

Kuberneteクライアントの[credential plugins](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins)のサポートは1.11からベータです。Kubernetes 1.22リリースではStableとなりました。GA機能セットは双方向のログイン処理を提供するプラグインについて改良されたサポートに加えて多くのバグフィックスを含んでいます。意欲的なプラグイン作者は[sample-exec-plugin](https://github.com/ankeesler/sample-exec-plugin)を見ることで開始することます。

### etcd moves to 3.5.0

Kubernetes' default backend storage, etcd, has a new release: 3.5.0. The new release comes with improvements to the security, performance, monitoring, and developer experience. There are numerous bug fixes and some critical new features like the migration to structured logging and built-in log rotation. The release comes with a detailed future roadmap to implement a solution to traffic overload. You can read a full and detailed list of changes in the [3.5.0 release announcement](https://etcd.io/blog/2021/announcing-etcd-3.5/).

Kubernetesのデフォルトのバックエンドストレージであるetcdの新しいリリース3.5.0があります。新リリースはセキュリティ、パフォーマンス、モニタリング、開発者の体験が向上しています。数多くのバグフィックスと構造化されたログ機能への以降やビルトインのログローテーションのような重要な新機能があります。このリリースはトラフィック負荷に対する解決策を実装するための詳細な未来のロードマップが付属しています。詳細な変更のリストは[3.5.0 release announcement](https://etcd.io/blog/2021/announcing-etcd-3.5/)から読むことが出来ます。

### Quality of Service for memory resources

Originally, Kubernetes used the v1 cgroups API. With that design, the QoS class for a `Pod` only applied to CPU resources (such as `cpu_shares`). As an alpha feature, Kubernetes v1.22 can now use the cgroups v2 API to control memory allocation and isolation. This feature is designed to improve workload and node availability when there is contention for memory resources, and to improve the predictability of container lifecycle.

元はKubernetesはv1のcgroup APIをつかっていました。そのデザインでは、`Pod` のQoSクラスはCPUリソース(`cpu_shares`など)のみに適用されています。Appha機能としてKubernetes v1.22ではcgroup v2 APIを使ったメモリの割当と隔離が可能です。この機能はメモリリソースの競合がある場合にワークロードとノードの可用性の向上と、コンテナのライフサイクルにおける予測可能性も向上させるよう設計されています。

### Node system swap support

Every system administrator or Kubernetes user has been in the same boat regarding setting up and using Kubernetes: disable swap space. With the release of Kubernetes 1.22, alpha support is available to run nodes with swap memory. This change lets administrators opt in to configuring swap on Linux nodes, treating a portion of block storage as additional virtual memory.

全てのシステム管理者やKubernetesユーザはKubernetesをセットアップしたり使う時、同じ環境にいます。つまりスワップスペースが無効になっています。Kubernetes 1.22リリースでは、Alpha機能としてスワップメモリを使用してノードを実行させることが出来ます。この変更では、管理者はLinuxノードでのスワップ構成を選択でき、ブロックストレージの一部分を追加の仮想メモリとして取り扱います。

### Windows enhancements and capabilities

Continuing to support the growing developer community, SIG Windows has released their [Development Environment](https://github.com/kubernetes-sigs/sig-windows-dev-tools/). These new tools support multiple CNI providers and can run on multiple platforms. There is also a new way to run bleeding-edge Windows features from scratch by compiling the Windows kubelet and kube-proxy, then using them along with daily builds of other Kubernetes components.

成長する開発者コミュニティを継続的にサポートするため、SIG Windowsは[Development Environment](https://github.com/kubernetes-sigs/sig-windows-dev-tools/)をリリースしました。これらの新しいツールは複数のCNIプロバイダをサポートし、複数のプラットフォームで実行することが出来ます。また、Windowsのkubeletとkube-proxyをコンパイルし、デイリービルドのKubernetesコンポーネントと一緒に使うことで、最先端のWindowsの機能を最初から実行する新しい方法もあります。

CSI support for Windows nodes moves to GA in the 1.22 release. In Kubernetes v1.22, Windows privileged containers are an alpha feature. To allow using CSI storage on Windows nodes, [CSIProxy](https://github.com/kubernetes-csi/csi-proxy) enables CSI node plugins to be deployed as unprivileged pods, using the proxy to perform privileged storage operations on the node.

WindowsノードのCSIサポートは1.22リリースでGAとなります。Kubernetes v1.22では、Windowsの特権コンテナはAlpha機能です。CSIストレージをWindowsノードでの使用を許可するため、[CSIProxy](https://github.com/kubernetes-csi/csi-proxy)を使うとCSIノードプラグインを非特権Podとしてデプロイできます。プロキシを使うことで、ノードにおける特権のストレージ操作を実行します。

### Default profiles for seccomp

An alpha feature for default seccomp profiles has been added to the kubelet, along with a new command line flag and configuration. When in use, this new feature provides cluster-wide seccomp defaults, using the `RuntimeDefault` seccomp profile rather than `Unconfined` by default. This enhances the default security of the Kubernetes Deployment. Security administrators will now sleep better knowing that workloads are more secure by default. To learn more about the feature, please refer to the official [seccomp tutorial](https://kubernetes.io/docs/tutorials/clusters/seccomp/#enable-the-use-of-runtimedefault-as-the-default-seccomp-profile-for-all-workloads).

デフォルトのseccompプロファイルのためのAlpha機能がKubeletに新しいコマンドラインのフラグとコンフィグとともに追加されました。使用すると、この新しい機能はクラスタ全体のseccompプロファイルを提供し、デフォルトでは`Unconfined`ではなく`RuntimeDefault`seccompプロファイルが使用されます。これはデフォルトのKubernetesのDeploymentのセキュリティを強化します。セキュリティ管理者はデフォルトでよりセキュアなワークロードが実行されるため安心できます。機能詳細を学ぶには公式の[seccomp tutorial](https://kubernetes.io/docs/tutorials/clusters/seccomp/#enable-the-use-of-runtimedefault-as-the-default-seccomp-profile-for-all-workloads)を参照してください。

### More secure control plane with kubeadm

A new alpha feature allows running the `kubeadm` control plane components as non-root users. This is a long requested security measure in `kubeadm`. To try it you must enable the `kubeadm` specific RootlessControlPlane feature gate. When you deploy a cluster using this alpha feature, your control plane runs with lower privileges.

新しいAlpha機能として`kubeadm`のコントロールプレーンを非rootユーザで実行することが可能になります。これは`kubeadm`で長くリクエストされていたセキュリティ対策です。これを試すためには`kubeadm`特有のRootlessControlPlaneのフィーチャーゲートを有効にする必要があります。このAlpha機能を使ったクラスタをデプロイする時、コントロールプレーンは低い権限で実行されます。

For `kubeadm`, Kubernetes 1.22 also brings a new [v1beta3 configuration API](https://github.com/kubernetes/kubeadm/issues/1796). This iteration adds some long requested features and deprecates some existing ones. The v1beta3 version is now the preferred API version; the v1beta2 API also remains available and is not yet deprecated.

`kubeadm`のため、Kubernetes 1.22は新しい[v1beta3 configuration API](https://github.com/kubernetes/kubeadm/issues/1796)も導入しました。この開発の中でいくつかの長期にリクエストされていた機能やいくつかの既存機能を非推奨としました。v1beta3バージョンは現在の優先APIバージョンです。v1beta2 APIは引き続き利用可能であり、まだ非推奨ではありません。

## Major Changes

### Removal of several deprecated beta APIs

A number of deprecated beta APIs have been removed in 1.22 in favor of the GA version of those same APIs. All existing objects can be interacted with via stable APIs. This removal includes beta versions of the `Ingress`, `IngressClass`, `Lease`, `APIService`, `ValidatingWebhookConfiguration`, `MutatingWebhookConfiguration`, `CustomResourceDefinition`, `TokenReview`, `SubjectAccessReview`, and `CertificateSigningRequest` APIs.

非推奨なBeta APIの大部分は1.22では削除され、同じAPIのGAバージョンが採用されました。存在している全てのオブジェクトは安定版APIを使って対話できます。削除されたものの中にはBetaバージョンの`Ingress`, `IngressClass`, `Lease`, `APIService`, `ValidatingWebhookConfiguration`, `MutatingWebhookConfiguration`, `CustomResourceDefinition`, `TokenReview`, `SubjectAccessReview`, `CertificateSigningRequest` APIが含まれます。

For the full list, check out the [Deprecated API Migration Guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#v1-22) as well as the blog post [Kubernetes API and Feature Removals In 1.22: Here’s What You Need To Know](https://blog.k8s.io/2021/07/14/upcoming-changes-in-kubernetes-1-22/). 

完全なリストとしては、[Deprecated API Migration Guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#v1-22)をチェックしてください。ブログの[Kubernetes API and Feature Removals In 1.22: Here’s What You Need To Know](https://blog.k8s.io/2021/07/14/upcoming-changes-in-kubernetes-1-22/)もチェックしてください。

###  API changes and improvements for ephemeral containers

The API used to create [Ephemeral Containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) changes in 1.22. The Ephemeral Containers feature is alpha and disabled by default, and the new API does not work with clients that attempt to use the old API.

[Ephemeral Containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)を作るためのAPIが1.22では変更されています。Ephemeral Container機能はAplhaでありデフォルトでは無効化されています。また、新APIは古いAPIを使うクライアントでは機能しません。

For stable features, the kubectl tool follows the Kubernetes [version skew policy](https://kubernetes.io/releases/version-skew-policy/); however, kubectl v1.21 and older do not support the new API for ephemeral containers. If you plan to use `kubectl debug` to create ephemeral containers, and your cluster is running Kubernetes v1.22, you cannot do so with kubectl v1.21 or earlier. Please update kubectl to 1.22 if you wish to use `kubectl debug` with a mix of cluster versions.

安定機能として、kubectlツールはKubernetes [version skew policy](https://kubernetes.io/releases/version-skew-policy/)に従います。しかし、kubectl v1.21 とそれより古いものは新しいEphemeral ContainersのAPIをサポートしません。Ephemeral Containerを作るために`kubectl debug`を使う予定があり、かつクラスターがKubernetes v1.22で実行している場合、kubectl v1.21 やそれよりも前のものでは実行できません。もしクラスターのバージョンを組み合わせて`kubectl debug`を使いたい場合は、kubectl を1.22にアップデートしてください。

## Other Updates

### Graduated to Stable

* [Bound Service Account Token Volumes](https://github.com/kubernetes/enhancements/issues/542)
* [CSI Service Account Token](https://github.com/kubernetes/enhancements/issues/2047)
* [Windows Support for CSI Plugins](https://github.com/kubernetes/enhancements/issues/1122)
* [Warning mechanism for deprecated API use](https://github.com/kubernetes/enhancements/issues/1693)
* [PodDisruptionBudget Eviction](https://github.com/kubernetes/enhancements/issues/85)

※ここは特に翻訳していません。

### Notable Feature Updates

* A new [PodSecurity admission](https://github.com/kubernetes/enhancements/issues/2579) alpha feature is introduced, intended as a replacement for PodSecurityPolicy
* [The Memory Manager](https://github.com/kubernetes/enhancements/issues/1769) moves to beta
* A new alpha feature to enable [API Server Tracing](https://github.com/kubernetes/enhancements/issues/647)
* A new v1beta3 version of the [kubeadm configuration](https://github.com/kubernetes/enhancements/issues/970) format
* [Generic data populators](https://github.com/kubernetes/enhancements/issues/1495) for PersistentVolumes are now available in alpha
* The Kubernetes control plane will now always use the [CronJobs v2 controller](https://github.com/kubernetes/enhancements/issues/19)
* As an alpha feature, all Kubernetes node components (including the kubelet, kube-proxy, and container runtime) can be [run as a non-root user](https://github.com/kubernetes/enhancements/issues/2033)

* 新規の[PodSecurity admission](https://github.com/kubernetes/enhancements/issues/2579)がAlpha機能として導入されました。これはPodSecurityPolicyの置き換えとして考えられています。
* [The Memory Manager](https://github.com/kubernetes/enhancements/issues/1769)がBetaとなりました。
* 新規のAlpha機能として[API Server Tracing](https://github.com/kubernetes/enhancements/issues/647)を有効に出来ます。
* [kubeadm configuration](https://github.com/kubernetes/enhancements/issues/970)のフォーマットが新しくv1beta3バージョンとなりました。
* PersistentVolumeの[Generic data populators](https://github.com/kubernetes/enhancements/issues/1495)がAlphaとして利用可能です。
* Kubernetesのコントロールプレーンは常に[CronJobs v2 controller](https://github.com/kubernetes/enhancements/issues/19)を使うようになります。
* 新規のAlpha機能として、全てのKubernetesノードコンポーネント(kubelet, kube-proxy, container runtimeを含みます)は[非root ユーザによって](https://github.com/kubernetes/enhancements/issues/2033)実行できます。

# Release notes

You can check out the full details of the 1.22 release in the [release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md).

1.22リリースの完全な詳細は[release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.22.md)でチェックできます。

# Availability of release

Kubernetes 1.22 is [available for download](https://kubernetes.io/releases/download/) and also [on the GitHub project](https://github.com/kubernetes/kubernetes/releases/tag/v1.22.0)). 

Kubernetes 1.22 は[ダウンロード可能](https://kubernetes.io/releases/download/)で、[GitHubのプロジェクト](https://github.com/kubernetes/kubernetes/releases/tag/v1.22.0)でも入手可能です。

There are some great resources out there for getting started with Kubernetes. You can check out some [interactive tutorials](https://kubernetes.io/docs/tutorials/) on the main Kubernetes site, or run a local cluster on your machine using Docker containers with [kind](https://kind.sigs.k8s.io). If you’d like to try building a cluster from scratch, check out the [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) tutorial by Kelsey Hightower.

Kubernetewを始める際の素晴らしいリソースがあります。メインのKubernetesのサイトでいくつかの[対話型のチュートリアル](https://kubernetes.io/docs/tutorials/)がチェックできます。あるいは、[kind](https://kind.sigs.k8s.io)により、Dockerコンテナを使ったマシン上でローカルクラスタを実行可能です。はじめからクラスタを構築してみたい場合は、Kelsey Hightowerによる[Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)のチュートリアルをチェックしてください。

# Release Team

This release was made possible by a very dedicated group of individuals, who came together as a team to deliver technical content, documentation, code, and a host of other components that go into every Kubernetes release.

このリリースは大変献身的な個人のグループによって作るのが可能になりました。このグループは全てのKuberneteのリリースに含まれる技術的な内容やドキュメンテーション、コード、その他のコンポーネントのホストを提供するチームです。

A huge thank you to the release lead Savitha Raghunathan for leading us through a successful release cycle, and to everyone else on the release team for supporting each other, and working so hard to deliver the 1.22 release for the community.

リリースサイクルを成功に導いてくれたリリースリードであるSavitha Raghunathanと、お互いにサポートしあったリリースチームのその他全員、コミュニティへ1.22リリースを提供するために一生懸命作業をしてくれたことに心から感謝します。

We would also like to take this opportunity to remember Peeyush Gupta, a member of our team that we lost earlier this year. Peeyush was actively involved in SIG ContribEx and the Kubernetes Release Team, most recently serving as the 1.22 Communications lead. His contributions and efforts will continue to reflect in the community he helped build. A [CNCF memorial](https://github.com/cncf/memorials/blob/main/peeyush-gupta.md) page has been created where thoughts and memories can be shared by the community.

私達はここで今年のはじめに失った我々のチームメンバーであるPeeyush Guptaを思い出したいと思います。PeeyushはSIG ContribExとKubernetesリリースチームで積極的に活動していました。最近の1.22におけるコミュニケーションのリードでした。彼の貢献と努力は彼が構築するのを助けたコミュニティに反映され続けるでしょう。[CNCF memorial](https://github.com/cncf/memorials/blob/main/peeyush-gupta.md)のページが作られ、コミュニティが考えや思い出を共有できるようになりました。

# Release Logo

![Kubernetes 1.22 Release Logo](https://d33wubrfki0l68.cloudfront.net/321bc629955ef59c388686099bc152a1386c4b34/ac495/images/blog/2021-08-04-kubernetes-release-1.22/kubernetes-1.22.png)

Amidst the ongoing pandemic, natural disasters, and ever-present shadow of burnout, the 1.22 release of Kubernetes includes 53 enhancements. This makes it the largest release to date. This accomplishment was only made possible due to the hard-working and passionate Release Team members and the amazing contributors of the Kubernetes ecosystem. The release logo is our reminder to keep reaching for new milestones and setting new records. And it is dedicated to all the Release Team members, hikers, and stargazers!

継続するパンデミック、自然災害、常に存在する燃え尽き症候群の影という状況の中で、Kubernetesの1.22リリースは53個のEnhancementが出来ました。このことはこれまでで最大のリリースとなります。この業績はハードな作業と熱意のあるリリースチームメンバーとKubernetesエコシステムの素晴らしいコントリビュータ達によってのみ実現可能でした。リリースロゴは新しいマイルストーンに到達し続けること、そして新しい記録を打ち立て続けるための我々のリマインダーです。そして全てのリリースチームメンバー、ハイカー、スターゲイバーに捧げられています。

The logo is designed by [Boris Zotkin](https://www.instagram.com/boris.z.man/). Boris is a Mac/Linux Administrator at the MathWorks. He enjoys simple things in life and loves spending time with his family. This tech-savvy individual is always up for a challenge and happy to help a friend!

ロゴは[Boris Zotkin](https://www.instagram.com/boris.z.man/)によってデザインされました。BorisはMathWorksのMac/Linuxの管理者です。彼は人生のシンプルさを楽しみ、家族との時間を過ごすのが大好きです。技術に精通した彼は常にチャレンジを続け友人を助けることを幸せに思います。

# User Highlights

- In May, the CNCF welcomed 27 new organizations across the globe as members of the diverse cloud native ecosystem. These new [members](https://www.cncf.io/announcements/2021/05/05/27-new-members-join-the-cloud-native-computing-foundation/) will participate in CNCF events, including the upcoming [KubeCon + CloudNativeCon NA in Los Angeles](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/) from October 12 – 15, 2021.
- The CNCF granted Spotify the [Top End User Award](https://www.cncf.io/announcements/2021/05/05/cloud-native-computing-foundation-grants-spotify-the-top-end-user-award/) during [KubeCon + CloudNativeCon EU – Virtual 2021](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/).

- ５月にCNCFは世界中の27の新しい組織を多様なクラウドネイティブエコシステムのメンバーとして迎えました。これらの新しい[メンバー](https://www.cncf.io/announcements/2021/05/05/27-new-members-join-the-cloud-native-computing-foundation/)は、来る2021/10/12 - 15の[KubeCon + CloudNativeCon NA in Los Angeles](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/)を含むCNCFのイベントに参加するでしょう。
- CNCFは[KubeCon + CloudNativeCon EU – Virtual 2021](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)の中でSpotifyに[Top End User Award](https://www.cncf.io/announcements/2021/05/05/cloud-native-computing-foundation-grants-spotify-the-top-end-user-award/) を与えました。

# Project Velocity

The [CNCF K8s DevStats project](https://k8s.devstats.cncf.io/) aggregates a number of interesting data points related to the velocity of Kubernetes and various sub-projects. This includes everything from individual contributions to the number of companies that are contributing, and is an illustration of the depth and breadth of effort that goes into evolving this ecosystem.

[CNCF K8s DevStats project](https://k8s.devstats.cncf.io/)では、Kubernetesやそのサブプロジェクトのベロシティに関する興味深いデータの数字を集約します。ここには個人のコントリビュータからコントリビューションしている企業の数まで含まれており、このエコシステムを進化させる努力の深さと広さを示しています。

In the v1.22 release cycle, which ran for 15 weeks (April 26 to August 4), we saw contributions from [1063 companies](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.21.0%20-%20now&var-metric=contributions) and [2054 individuals](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.21.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All).

v1.22リリースサイクルは15週間(04/26 - 08/04)でした。[1063の企業](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.21.0%20-%20now&var-metric=contributions) と [2054の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.21.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All)によるコントリビューションがありました。

# Ecosystem Updates

- [KubeCon + CloudNativeCon Europe 2021](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/) was held in May, the third such event to be virtual. All talks are [now available on-demand](https://www.youtube.com/playlist?list=PLj6h78yzYM2MqBm19mRz9SYLsw4kfQBrC) for anyone that would like to catch up!
- [Spring Term LFX Program](https://www.cncf.io/blog/2021/07/13/spring-term-lfx-program-largest-graduating-class-with-28-successful-cncf-interns) had the largest graduating class with 28 successful CNCF interns!
- CNCF launched [livestreaming on Twitch](https://www.cncf.io/blog/2021/06/03/cloud-native-community-goes-live-with-10-shows-on-twitch/) at the beginning of the year targeting definitive interactive media experience for anyone wanting to learn, grow, and collaborate with others in the Cloud Native community from anywhere in the world.

- [KubeCon + CloudNativeCon Europe 2021](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)が5月に開催されました。ヴァーチャルイベントの3回目であり、確認したい人は全てのトークを[オンデマンドで確認可能です。](https://www.youtube.com/playlist?list=PLj6h78yzYM2MqBm19mRz9SYLsw4kfQBrC)
- [Spring Term LFX Program](https://www.cncf.io/blog/2021/07/13/spring-term-lfx-program-largest-graduating-class-with-28-successful-cncf-interns)で28人のCNCFインターン生が卒業しました。
- CNCFは今年のはじめに世界のどこからでもクラウドネイティブコミュニティで他の人と学び、成長し、コラボレーションしたい全ての人に対する双方向のメディア体験をターゲットとして、[Twitchのライブストリーミング](https://www.cncf.io/blog/2021/06/03/cloud-native-community-goes-live-with-10-shows-on-twitch/)を開始しました。

# Event Updates

- [KubeCon + CloudNativeCon North America 2021](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/) will take place in Los Angeles, October 12 – 15, 2021! You can find more information about the conference and registration on the event site.
- [Kubernetes Community Days](https://community.cncf.io/kubernetes-community-days/about-kcd/) has upcoming events scheduled in Italy, the UK, and in Washington DC.

- [KubeCon + CloudNativeCon North America 2021](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/)が2021/10/12 - 15にロサンゼルスで開催されます!カンファレンスの詳細や登録については、イベントのサイトを参照してください。
- [Kubernetes Community Days](https://community.cncf.io/kubernetes-community-days/about-kcd/)がイタリア、UK、そしてワシントンDCで予定されています。

# Upcoming release webinar

Join members of the Kubernetes 1.22 release team on September 7, 2021 to learn about the major features of this release, as well as deprecations and removals to help plan for upgrades. For more information and registration, visit the [event page](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-122-release/) on the CNCF Online Programs site.

2021/09/07にKubernetes1.22のリリースチームメンバーがこのリリースの主要な機能やアップグレード計画を助けるための非推奨、削除機能について学ぶために(リリースウェビナーに)参加します。詳細情報と登録のためには、CNCF Online Programsサイトの[イベントページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-122-release/) を参照してください。

# Get Involved

If you’re interested in contributing to the Kubernetes community, Special Interest Groups (SIGs) are a great starting point. Many of them may align with your interests! If there are things you’d like to share with the community, you can join the weekly community meeting, or use any of the following channels:

もしKubernetesコミュニティへのコントリビューションに興味がある場合は、Special Interest Groups(SIGs)が出発地点として良いでしょう。SIGsの多くはあなたの興味に沿っていいることでしょう。もしコミュニティと共有したいことがある場合は、週次のコミュニティミーティングに参加することも可能ですし、以下のチャンネルも活用してください。

* Find out more about contributing to Kubernetes at the [Kubernetes Contributors](https://www.kubernetes.dev/) website.
* Follow us on Twitter [@Kubernetesio](https://twitter.com/kubernetesio) for latest updates
* Join the community discussion on [Discuss](https://discuss.kubernetes.io/)
* Join the community on [Slack](http://slack.k8s.io/)
* Share your Kubernetes [story](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)
* Read more about what’s happening with Kubernetes on the [blog](https://kubernetes.io/blog/)
* Learn more about the [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)

* [Kubernetes Contributors](https://www.kubernetes.dev/)でコントリビューションについての詳細を見つけられます。
* 最新の更新についてはTwitterの[@Kubernetesio](https://twitter.com/kubernetesio)をフォローしてください。
* コミュニティでの相談は[Discuss](https://discuss.kubernetes.io/)に参加してください。
* [Slack](http://slack.k8s.io/)のコミュニティに参加してください。
* あなたのKubernetesの[ストーリー](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)を共有してください。
* [blog](https://kubernetes.io/blog/)でKubernetesで起きてる出来事を読むことが出来ます。
* [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)についての詳細を知ることが出来ます。

