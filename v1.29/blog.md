本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

原文: https://kubernetes.io/blog/2023/12/13/kubernetes-v1-29-release/

**Authors:** [Kubernetes v1.29 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.29/release-team.md)

**Editors:** Carol Valencia, Kristin Martin, Abigail McCarthy, James Quigley

2023年最後のリリースとなるKubernetes v1.29: Mandala (The Universe)のリリースを発表します！

Kubernetes v1.29のリリースでは、これまでのリリースと同様に、安定版、ベータ版、アルファ版の新機能が導入されています。一流のリリースが一貫して提供されていることは、私たちの開発サイクルの強さとコミュニティからの活発なサポートを裏付けています。

このリリースは49の機能強化で構成されています。このうち11の機能拡張がStableに、19がBetaに、19がAlphaに移行しました。

## Release theme and logo
    
Kubernetes v1.29: *Mandala (The Universe)* ✨🌌

![Kubernetes 1.29 Mandala logo](https://kubernetes.io/images/blog/2023-12-13-kubernetes-1.29-release/k8s-1.29.png)

Kubernetes v1.29で宇宙の旅に出かけましょう！

このリリースは、宇宙の完璧さを象徴する美しい芸術である曼荼羅にインスパイアされています。約40人のリリースチームメンバーと数百人のコミュニティ貢献者からなる私たちの緊密な結束は、世界中の何百万人もの人々の挑戦を喜びに変えるため、たゆまぬ努力を続けてきました。

曼荼羅のテーマは、私たちのコミュニティの相互の結びつきを反映しています。各コントリビューターは、曼荼羅アートの多様な模様のように、独自のエネルギーを加える重要な役割を担っています。Kubernetesはコラボレーションによって繁栄し、曼荼羅の作品に見られる調和を再現しています。

リリースロゴは[Mario Jason Braganza](https://janusworx.com)によるもので、(ベースとなる曼荼羅アートは[Fibrel Ojalá](https://pixabay.com/users/fibrel-3502541/))Kubernetesプロジェクトとそのすべての人々である小さな宇宙を象徴しています。

曼荼羅の変容の象徴の精神に基づき、Kubernetes v1.29は私たちのプロジェクトの進化を祝福します。Kubernetesの宇宙の星のように、それぞれの貢献者、ユーザー、サポーターが道を照らします。共に、私たちは可能性の宇宙を創造します。

## Improvements that graduated to stable in Kubernetes v1.29 {#graduations-to-stable}

_これは、v1.29リリース後に安定版の改良点の一部です。_

### ReadWriteOncePod PersistentVolume access mode ([SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage)) 

Kubernetesでは、ボリュームの[アクセスモード](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
は、耐久性のあるストレージがどのように消費されるかを定義する方法です。これらのアクセスモードは、PersistentVolumes（PV）とPersistentVolumeClaims（PVC）の仕様の一部です。ストレージを使用する場合、そのストレージがどのように消費されるかをモデル化する方法はさまざまです。たとえば、ネットワークファイル共有のようなストレージシステムでは、多数のユーザが同時にデータを読み書きすることができます。また、誰もがデータを読むことはできるが、書き込むことはできない場合もあります。機密性の高いデータの場合、一人のユーザーだけがデータの読み書きを許可され、他は誰も許可されないこともあります。

v1.22以前のKubernetesでは、PVとPVCに対して3つのアクセスモードが提供されていました。
* ReadWriteOnce - ボリュームを1つのノードで読み書き可能にマウントできます。
* ReadOnlyMany - ボリュームを多数のノードで読み取り専用にマウントできます。
* ReadWriteMany - ボリュームを多数のノードで読み書き可能にマウントできます。

ReadWriteOnceアクセスモードでは、ボリュームアクセスが単一のノードに制限されるため、同じノード上の複数のポッドが同じボリュームから読み取り、同じボリュームに書き込むことが可能です。これは、特にデータの安全性を保証するために最大1つのライターを必要とする場合、アプリケーションによっては大きな問題になる可能性があります。
    
この問題に対処するため、CSIボリューム用のv1.22のアルファ機能として、4番目のアクセスモードReadWriteOncePodが導入されました。ReadWriteOncePodアクセスモードを使用するPVCでPodを作成すると、KubernetesはそのPodがクラスタ全体でそのPVCを読み取ったり書き込んだりできる唯一のPodであることを保証します。v1.29では、この機能が一般的に利用可能になりました。

### Node volume expansion Secret support for CSI drivers ([SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage)) 

Kubernetesでは、ボリューム拡張操作には、ファイルシステムのリサイズを伴うノード上のボリュームの拡張が含まれる場合があります。一部のCSIドライバは、以下のユースケースのために、ノードの拡張中にシークレット、例えばSANファブリックにアクセスするためのクレデンシャルを必要とします。
* PersistentVolume が LUKS などの暗号化されたブロックストレージを表す場合、デバイスを拡張するためにパスフレーズを提供する必要があります。
* 様々な検証のために、CSIドライバは、ノード拡張時にバックエンドストレージシステムと通信するための認証情報が必要です。

この要件を満たすために、CSI Node Expand Secret機能がKubernetes v1.25で導入されました。これにより、CSIドライバがNodeExpandVolumeRequestの一部としてオプションのシークレットフィールドを送信し、基礎となるストレージシステムでノードボリューム拡張操作を実行できるようになります。Kubernetes v1.29では、この機能は一般的に利用できるようになりました。

### KMS v2 encryption at rest generally available ([SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth))

Kubernetesクラスタのセキュリティを確保する際に最初に考慮すべきことの1つは、永続化されたAPIデータを暗号化することです。
KMSは、プロバイダが外部のキーサービスに保存されたキーを利用してこの暗号化を実行するためのインタフェースを提供します。
Kubernetes v1.29で、KMS v2は安定した機能となり、パフォーマンス、キーローテーションヘルスチェックとステータス、そして観測可能性において多くの改善がもたらされました。
これらの機能強化は、Kubernetesクラスタ内のすべてのリソースを暗号化する信頼性の高いソリューションをユーザーに提供します。これについては[KEP-3299](https://kep.k8s.io/3299)を参照してください。

KMS v2の利用を推奨します。KMS v1のフィーチャーゲートはデフォルトで無効になっています。引き続き使用するにはオプトインする必要があります。

## Improvements that graduated to beta in Kubernetes v1.29 

_v1.29のリリース後、現在ベータ版となっている改良点の一部をご紹介します。_

スケジューラのスループットは私たちの永遠の課題です。このQueueingHint機能は、再キューの効率を最適化する新しい可能性をもたらし、無駄なスケジューリングの再試行を大幅に減らすことができます。

### Node lifecycle separated from taint management ([SIG Scheduling](https://github.com/kubernetes/community/tree/master/sig-scheduling))

タイトルの通り、テイントベースのポッド削除を行う `TaintManager` と `NodeLifecycleController` を分離し、2つのコントローラにします。正常ではないノードにテイントを追加する `NodeLifecycleController` と、NoExecute エフェクトでテイントが付与されたノードのPod削除を行う `TaintManager` です。

### Clean up for legacy Secret-based ServiceAccount tokens ([SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth)) 

Kubernetesは、1.22で時間制限付きで特定のPodにバインドされた、よりセキュアなサービスアカウントトークンの使用に切り替えました。1.24でレガシーなシークレットベースのサービスアカウントトークンの自動生成を停止しています。そして1.27で、まだ使用されている残りの自動生成されたシークレットベースのトークンに、最終使用日のラベル付けを開始しました。

v1.29では、潜在的な攻撃対象領域を減らすために、LegacyServiceAccountTokenCleanUp機能は、レガシー自動生成されたシークレットベースのトークンが長期間（デフォルトで1年間）使用されていない場合、無効であるとラベル付けし、無効であるとマークされた後、長期間（デフォルトでさらに1年間）使用されない場合、自動的に削除します。[KEP-2799](https://kep.k8s.io/2799)
## New alpha features

### Define Pod affinity or anti-affinity using `matchLabelKeys` ([SIG Scheduling](https://github.com/kubernetes/community/tree/master/sig-scheduling))

アルファ版として、PodAffinity/PodAntiAffinityに1つの機能強化が導入されます。これにより、ローリングアップデート時の計算精度が向上します。

### nftables backend for kube-proxy ([SIG Network](https://github.com/kubernetes/community/tree/master/sig-network))
    
Linux上のデフォルトのkube-proxy実装は、現在iptablesに基づいている。iptablesは、Linuxカーネルで長年（2001年の2.4カーネルから）使用されてきたパケットフィルタリングおよび処理システムです。しかし、iptablesの解決できない問題により、後継のnftablesが開発されました。iptablesの開発はほとんど停止しており、新機能や性能の向上は主にnftablesで行われています。

いくつかのLinuxディストリビューションはすでにiptablesの非推奨と削除を開始しており、nftablesはiptablesの主なパフォーマンス問題を解決すると言われているため、この機能はnftablesをベースにkube-proxyに新しいバックエンドを追加します。
    
### APIs to manage IP address ranges for Services ([SIG Network](https://github.com/kubernetes/community/tree/master/sig-network)) 

Serviceは、一連のPod上で動作するアプリケーションを公開する抽象的な方法です。Serviceはクラスタスコープの仮想IPアドレスを持つことができ、これはkube-apiserverフラグで定義された定義済みのCIDRから割り当てられます。しかし、ユーザーはkube-apiserverを再起動することなく、サービスに割り当てられた既存のIP範囲を追加、削除、またはサイズ変更したい場合があります。

この機能は、2つの新しいAPIオブジェクトを使用する新しい割当ロジックを実装しています： ServiceCIDRとIPAddressの2つの新しいAPIオブジェクトを使用する新しい割当ロジックを実装しており、ユーザーは新しいServiceCIDRを作成することで、利用可能なサービスIPの数を動的に増やすことができます。これにより、IP枯渇やIPの再ナンバリングなどの問題を解決できます。

### Add support to containerd/kubelet/CRI to support image pull per runtime class ([SIG Windows](https://github.com/kubernetes/community/tree/master/sig-windows))

Kubernetes v1.29では、コンテナイメージを使用するPodのRuntimeClassに基づいてコンテナイメージをプルするサポートが追加された。
この機能は v1.29 では `RuntimeClassInImageCriApi` という機能ゲートでデフォルトではオフになっています。

コンテナイメージはマニフェストまたはインデックスのいずれかになります。プルされるイメージがインデックスである場合（イメージインデックスはプラットフォーム順に並べられたイメージマニフェストのリストを持ちます）、コンテナランタイムのプラットフォームのマッチングロジックはインデックスから適切なイメージマニフェストをプルするために使用されます。デフォルトでは、プラットフォームのマッチングロジックは、イメージのプルが実行されるホストに一致するマニフェストを選択します。これは、たとえば Windows Hyper-V コンテナなど、ユーザーが VM ベースのコンテナとして実行することを意図してイメージをプルする可能性があるVMベースのコンテナでは制限になる可能性があります。

ランタイムクラスごとのイメージプル機能は、指定されたランタイムクラスに基づいて異なるイメージをプルするサポートを追加します。これは `imageName` や `imageID` ではなく、(`imageID`, `runtimeClass`) のタプルでイメージを参照することで実現されます。コンテナランタイムはこの機能のサポートを追加することができます。追加しない場合は、Kubernetes v1.29以前のkubeletのデフォルトの動作が維持されます。

### In-place updates for Pod resources, for Windows Pods ([SIG Windows](https://github.com/kubernetes/community/tree/master/sig-windows))

アルファ機能として、KubernetesのPodはその`resources`に関してミュータブルにすることができ、ユーザーはPodを再起動することなく、Podの _希望された_ リソースと制限に変更することができます。v1.29では、この機能がWindowsコンテナでサポートされるようになりました。

## Graduations, deprecations and removals for Kubernetes v1.29

### Graduated to stable

これは、安定版（一般提供版とも呼ばれる）に移行したすべての機能のリストです。
新機能やアルファ版からベータ版への移行を含むアップデートの全リストは
[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.29.md) を参照してください。

このリリースには、Stable に昇格した合計 11 の機能強化が含まれています。

- [Remove transient node predicates from KCCM's service controller](https://kep.k8s.io/3458)
- [Reserve nodeport ranges for dynamic and static allocation](https://kep.k8s.io/3668)
- [Priority and Fairness for API Server Requests](https://kep.k8s.io/1040)
- [KMS v2 Improvements](https://kep.k8s.io/3299)
- [Support paged LIST queries from the Kubernetes API](https://kep.k8s.io/365)
- [ReadWriteOncePod PersistentVolume Access Mode](https://kep.k8s.io/2485)
- [Kubernetes Component Health SLIs](https://kep.k8s.io/3466)
- [CRD Validation Expression Language](https://kep.k8s.io/2876)
- [Introduce nodeExpandSecret in CSI PV source](https://kep.k8s.io/3107)
- [Track Ready Pods in Job status](https://kep.k8s.io/2879)
- [Kubelet Resource Metrics Endpoint](https://kep.k8s.io/727)

### Deprecations and removals

#### Removal of in-tree integrations with cloud providers ([SIG Cloud Provider](https://github.com/kubernetes/community/tree/master/sig-cloud-provider))

Kubernetes v1.29のデフォルトでは、クラウドプロバイダーとのビルトイン統合なしで動作します。
これまで（Azure、GCE、またはvSphereとの）ツリー内のクラウドプロバイダー統合に依存していた場合は、次のいずれかを行うことができます。
- 同等の外部[cloud controller manager](https://kubernetes.io/docs/concepts/architecture/cloud-controller/)統合を有効にする。_(推奨)_
- 関連する機能ゲートを `false` に設定することで、レガシーな統合に戻る。
  変更する機能ゲートは `DisableCloudProviders` と `DisableKubeletCloudCredentialProviders` です。
  
外部クラウドコントローラマネージャを有効にするということは、クラスタのコントロールプレーン内で適切なクラウドコントローラマネージャを実行する必要があるということです。また、kubelet（関連するすべてのノード）、およびコントロールプレーン全体（kube-apiserverとkube-controller-manager）でコマンドライン引数`--cloud-provider=external`を設定する必要があります。

外部クラウドコントローラマネージャを有効にして実行する方法の詳細については、[Cloud Controller Manager Administration](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/) および [Migrate Replicated Control Plane To Use Cloud Controller Manager](https://kubernetes.io/docs/tasks/administer-cluster/controller-manager-leader-migration/) を参照してください。

レガシーなインツリープロバイダー用のクラウドコントローラーマネージャーが必要な場合は、以下のリンクを参照してください： 
* [Cloud provider AWS](https://github.com/kubernetes/cloud-provider-aws)
* [Cloud provider Azure](https://github.com/kubernetes-sigs/cloud-provider-azure)
* [Cloud provider GCE](https://github.com/kubernetes/cloud-provider-gcp)
* [Cloud provider OpenStack](https://github.com/kubernetes/cloud-provider-openstack)
* [Cloud provider vSphere](https://github.com/kubernetes/cloud-provider-vsphere)

詳細は[KEP-2395](https://kep.k8s.io/2395)にあります。

#### Removal of the `v1beta2` flow control API group

非推奨の _flowcontrol.apiserver.k8s.io/v1beta2_ APIバージョンのFlowSchemaとPriorityLevelConfigurationは、Kubernetes v1.29では提供されなくなりました。

非推奨のベータAPIグループを使用するマニフェストまたはクライアントソフトウェアがある場合は、v1.29にアップグレードする前にこれらを変更する必要があります。
詳細とアドバイスについては、[deprecated API migration guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#v1-29) を参照してください。
 
#### Deprecation of the `status.nodeInfo.kubeProxyVersion` field for Node

Nodeオブジェクトの`.status.kubeProxyVersion`フィールドは現在非推奨となっており、Kubernetesプロジェクトは将来のリリースでこのフィールドを削除することを提案しています。非推奨のフィールドは正確ではなく、歴史的にkubeletによって管理されてきました。kubeletは実際にはkube-proxyのバージョンや、kube-proxyが実行されているかどうかさえ知りません。

クライアント・ソフトウェアでこのフィールドを使用していた場合は、使用を中止してください。情報は信頼できず、このフィールドは現在非推奨です。

#### Legacy Linux package repositories

2023年8月、レガシーパッケージリポジトリ（`apt.kubernetes.io`と`yum.kubernetes.io`）は正式に廃止され、KubernetesプロジェクトはDebianとRPMパッケージ用のコミュニティ所有のパッケージリポジトリ（`https://pkgs.k8s.io`で利用可能）の一般提供を発表しました。

これらのレガシーリポジトリは2023年9月に凍結され、2024年1月には完全になくなる予定です。現在これらのレガシーリポジトリに依存している場合、 **必ず** 移行してください。

この非推奨は v1.29 リリースとは直接関係ありません。_ この変更があなたにどのような影響を与えるか、また影響を受けた場合にどうすればよいかを含む詳細については、[レガシーパッケージリポジトリの非推奨に関するアナウンス](/blog/2023/08/31/legacy-package-repository-deprecation/) をお読みください。

## Release notes

Kubernetes v1.29リリースの詳細は[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.29.md)をご覧ください。

## Availability

Kubernetes v1.29は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.29.0)からダウンロードできます。Kubernetesを使い始めるには、以下の[インタラクティブチュートリアル](https://kubernetes.io/docs/tutorials)をチェックするか、[minikube](https://minikube.sigs.k8s.io/)を使ってローカルのKubernetesクラスタを実行してください。[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm)を使ってv1.29を簡単にインストールすることもできます。

## Release team

Kubernetesは、そのコミュニティのサポート、コミットメント、そしてハードワークがあって初めて実現可能になります。各リリースチームは、献身的なコミュニティのボランティアで構成されており、みなさんが信頼するKubernetesのリリースを構成する多くのパーツを協力して構築しています。これには、コードそのものからドキュメントやプロジェクト管理に至るまで、コミュニティのあらゆる場所にいる人々の専門的なスキルが必要です。

私たちのコミュニティのためにKubernetes v1.29リリースを提供するために懸命に働いてくれた[リリースチーム](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.29/release-team.md)全員に感謝したいと思います。私たちのリリースリードである[Priyanka Saggu](https://github.com/Priyankasaggu11929)には、リリースサイクルを成功に導き、私たち全員が可能な限り最善の方法で貢献できるようにサポートし、リリースプロセスを改善するために挑戦してくれたことに、特別な感謝を捧げます。

## Project velocity

CNCF K8s DevStatsプロジェクトは、Kubernetesと様々なサブプロジェクトのベロシティに関連する数多くの興味深いデータを集約しています。これには、個人のコントリビューションからコントリビューションしている企業の数まであらゆるものが含まれ、このエコシステムを進化させるために費やされている努力の深さと広さを示しています。

[14週間](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.29)(9月6日から12月13日まで)実施されたv1.29リリースサイクルでは、[888社](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.28.0%20-%20now&var-metric=contributions)と[1422人の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.28.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-repo_name=kubernetes%2Fkubernetes&var-country_name=All&var-companies=All)からの貢献がありました。


## Ecosystem updates

- KubeCon + CloudNativeCon Europe 2024は、**2024年3月19日 〜 22日**にフランスのパリで開催されます！[イベントサイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)で、カンファレンスの詳細と参加登録ができます。

## Upcoming release webinar

2023年12月15日（金）PT午前11時（東部時間午後2時）より、Kubernetes v1.29リリースチームのメンバーと一緒に、このリリースの主な機能、およびアップグレードの計画に役立つ非推奨機能や削除機能について学びましょう。詳細と参加登録は、CNCF Online Programsサイトの[イベントページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-129-release/)をご覧ください。

### Get involved

Kubernetesに参加する最も簡単な方法は、あなたの興味に沿った多くの[Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs)の1つに参加することです。Kubernetesコミュニティに発信したいことがありますか？私たちが毎週開催している[コミュニティ・ミーティング](https://github.com/kubernetes/community/tree/master/communication)や以下のチャンネルであなたの声を共有してください。引き続きフィードバックとサポートをよろしくお願いします。

- Twitter [@Kubernetesio](https://twitter.com/kubernetesio)で最新情報をフォローしてください。
- [Discuss](https://discuss.kubernetes.io/)でコミュニティの議論に参加する
- [Slack](http://slack.k8s.io/) でコミュニティに参加してください。
- [Stack Overflow](http://stackoverflow.com/questions/tagged/kubernetes)で質問を投稿する(または質問に答える)
- あなたのKubernetes [ストーリー](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)を共有する
- [ブログ](https://kubernetes.io/blog/)でKubernetesの最新情報を読む
- [Kubernetesリリースチーム](https://github.com/kubernetes/sig-release/tree/master/release-team)をもっとよく知る
