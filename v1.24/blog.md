本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

---

# Kubernetes 1.24: Stargazer

Original Post: https://kubernetes.io/blog/2022/05/03/kubernetes-1-24-release-announcement/

**Authors**: [Kubernetes 1.24 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.24/release-team.md)

2022年の最初のリリースであるKubernetes 1.24のリリースをアナウンスできることを嬉しく思います。

このリリースには46個のEnhancementがありました。14個がStableとなり、15個がBetaとなり、13個がAlphaとなりました。また、2個がDepracatedとなり、2個が削除されました。

## Major Themes

### Dockershim Removed from kubelet

v1.20における非推奨の後に、dockershim コンポーネントはv1.24で kubelet から削除されました。
v1.24以降では、（containerd や CRI-O のような）[サポートされた他のランタイム](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)の1つを使うか、コンテナランタイムとしてDocker Engineに依存している場合は cri-dockerd を使う必要があります。
クラスタが今回の削除に対して準備されているか確認するための詳細は、[このガイド](https://kubernetes.io/blog/2022/03/31/ready-for-dockershim-removal/)を参照してください。

### Beta APIs Off by Default

[新規のベータAPIはデフォルトではクラスタで有効になりません](https://github.com/kubernetes/enhancements/issues/3136)。
既存のベータAPIと既存のベータAPIの新しいバージョンはデフォルトで有効化されたままとなります。

### Signing Release Artifacts

リリース物は [cosign](https://github.com/sigstore/cosign) のシグネチャを使って[署名され](https://github.com/kubernetes/enhancements/issues/3031)、[イメージ署名の検証](https://kubernetes.io/docs/tasks/administer-cluster/verify-signed-images/)が実験的にサポートされています。
リリース物の署名と検証は、[Kubernetesリリースプロセスのソフトウェアサプライ・チェインセキュリティの強化](https://github.com/kubernetes/enhancements/issues/3027)の一部分として実施されています。

### OpenAPI v3

Kubernetes 1.24は、 [OpenAPI v3 フォーマット](https://github.com/kubernetes/enhancements/issues/2896)によるAPIのパブリッシュをベータサポートを提供します。

### Storage Capacity and Volume Expansion Are Generally Available

[ストレージ容量追跡(Storage capacity tracking)](https://github.com/kubernetes/enhancements/issues/1472) は[CSIStorageCapacityオブジェクト](https://kubernetes.io/docs/concepts/storage/storage-capacity/#api) を使うことで、現在の利用可能なストレージ容量の公開をサポートします。また、CSIボリュームを遅延バインドで使ったPodのスケジューリングを拡張します。

[ボリューム拡張](https://github.com/kubernetes/enhancements/issues/284)は、既存の永続ボリュームのリサイズのサポートを追加しました。

### NonPreemptingPriority to Stable

このフィーチャーは [PriorityClassesの新しい選択肢](https://github.com/kubernetes/enhancements/issues/902) を追加します。これによりPodのプリエンプションの有効化/無効化を行うことができます。

### Storage Plugin Migration

オリジナルのAPIを維持しながら、[インツリーのストレージプラグインの内部をマイグレーション](https://github.com/kubernetes/enhancements/issues/625)し、CSIプラグインをコールする作業が進行中です。
[Azure Disk](https://github.com/kubernetes/enhancements/issues/1490) と [OpenStack Cinder](https://github.com/kubernetes/enhancements/issues/1489) は両方とも移行が完了しています。

### gRPC Probes Graduate to Beta

Kubernetes 1.24では、[gRPC probes 機能](https://github.com/kubernetes/enhancements/issues/2727)がベータとなりデフォルトで利用可能です。gRPCアプリに対して [startup probes、 liveness probes および readiness probesの編集](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes)がHTTPエンドポイントを公開することや追加の実行を必要とせずKubernetes内でネイティブに利用可能です。

### Kubelet Credential Provider Graduates to Beta

元はKubernetes v1.20でアルファリリースされたもので、kubelet の[イメージクレデンシャルプロバイダ](https://kubernetes.io/docs/tasks/kubelet-credential-provider/kubelet-credential-provider/)がベータとなりました。
これにより kubelet はノードのファイルシステムにクレデンシャルを保管することなくプラグインを実行することによってコンテナイメージレジストリのクレデンシャルを動的に取得することができます。

### Contextual Logging in Alpha

Kubernetes 1.24は[コンテキストロギング(contextual logging)](https://github.com/kubernetes/enhancements/issues/3077) を導入しました。これは関数の呼び出し側がロギングの全ての側面(出力フォーマット、詳細度(verbosity)、追加の値、名前)を制御することを可能にします。

### Avoiding Collisions in IP allocation to Services

Kubernetes 1.24はServiceに対して[静的IPアドレス割り当ての範囲を緩やかに予約(soft-reserve)](https://kubernetes.io/docs/concepts/services-networking/service/#service-ip-static-sub-range)することのできる新たに選択可能な機能を導入します。
この機能を手動で有効化することで、クラスタは Service のIPアドレスプールから自動割当することを優先し(prefer)、衝突のリスクを減らすことになります。

Service の `ClusterIP` は次のように割り当て可能です。

* 動的割当は、クラスタが設定されたService IPの範囲から利用可能なIPを自動的に選択します。
* 静的割当は、ユーザが設定されたService IPの範囲から1つのIPを設定します。

Service の `ClusterIP` は唯一なものなので、すでに割当済みの `ClusterIP` で Service を作ろうとすると、エラーが返却されます。

### Dynamic Kubelet Configuration is Removed from the Kubelet

Kubernetes 1.22 で非推奨となって以降、動的なkubeletの設定(Dynamic Kubelet Cofiguration)機能は、kubelet から削除されました。この機能はKubernetes 1.26でAPIサーバから削除される予定です。

## CNI Version-Related Breaking Change

Kubernetes 1.24にアップグレードする前に、正常に動作することがテストされているコンテナランタイムを使っている/アップグレードしていることを検証してください。

例えば、次のコンテナランタイムは準備されているか、あるいはKubernetesに向けて既に準備されています。

* containerd v1.6.4以降、v1.5.11以降
* CRI-O 1.24以降

CNIプラグインがアップグレードされておらず、かつCNIの設定ファイルにCNI設定バージョンが定義されていない場合、containerd v1.6.0 〜 v1.6.3を用いた環境では、Serviceに関するPodのCNIネットワークの構築と後処理の問題が存在します。containerdチームは「これらの問題は containerd v1.6.4 で解決されています。」と報告しています。

containerd v1.6.0 〜 v1.6.3 では、もしもCNIプラグインをアップグレードしておらず、かつCNI設定バージョンが定義されていない場合、 "Incompatible
CNI versions" や "Failed to destroy network for sandbox" というエラー状態に遭遇するかも知れません。

## Other Updates

### Graduations to Stable

今回のリリースでは次の14個がStableになりました。

* [Container Storage Interface (CSI) Volume Expansion](https://github.com/kubernetes/enhancements/issues/284)
* [Pod Overhead](https://github.com/kubernetes/enhancements/issues/688): Podのサンドボックスに関連付けられているが、特定のコンテナには関連付けられていないリソースを考慮します。
* [Add non-preempting option to PriorityClasses](https://github.com/kubernetes/enhancements/issues/902)
* [Storage Capacity Tracking](https://github.com/kubernetes/enhancements/issues/1472)
* [OpenStack Cinder In-Tree to CSI Driver Migration](https://github.com/kubernetes/enhancements/issues/1489)
* [Azure Disk In-Tree to CSI Driver Migration](https://github.com/kubernetes/enhancements/issues/1490)
* [Efficient Watch Resumption](https://github.com/kubernetes/enhancements/issues/1904): kube-apiserver の再起動後にWatchは効率的に復帰できます。
* [Service Type=LoadBalancer Class Field](https://github.com/kubernetes/enhancements/issues/1959): 新しいServiceのアノテーションの`service.kubernetes.io/load-balancer-class` が導入されました。これは、同じクラスタ内で複数の実装の`type: LoadBalancer`のServicesを可能とします。
* [Indexed Job](https://github.com/kubernetes/enhancements/issues/2214): 完了インデックスがJobのPodに対して固定の完了カウントとして追加されました。
* [Add Suspend Field to Jobs API](https://github.com/kubernetes/enhancements/issues/2232): 一時停止のフィールドをJobs APIに追加し、オーケストレータによってPodが作られた時のジョブの細かい制御が可能となりました。
* [Pod Affinity NamespaceSelector](https://github.com/kubernetes/enhancements/issues/2249): `namespaceSelector` フィールドがPodの affinity/anti-affinity の仕様に追加されました。
* [Leader Migration for Controller Managers](https://github.com/kubernetes/enhancements/issues/2436): kube-controller-manager と cloud-controller-manager は、ダウンタイムなしにHA構成のコントロールプレーンで新しい controller-to-controller-manager の割当を適用可能です。
* [CSR Duration](https://github.com/kubernetes/enhancements/issues/2784): CertificateSigningRequest API を拡張し、クライアントが発行された証明書に対する特定の期限のリクエストが可能となりました。

### Major Changes

今回のリリースでは次の2つのMajor Changeがあります。

* [Dockershim Removal](https://github.com/kubernetes/enhancements/issues/2221)
* [Beta APIs are off by Default](https://github.com/kubernetes/enhancements/issues/3136)

### Release Notes

Kubernetes 1.24 リリースの完全な詳細はこちらの [リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md) をチェックしてください。

### Availability

Kubernetes 1.24 は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.24.0)からダウンロード可能です。
Kubernetesを始めるには、こちらの[対話型チュートリアル](https://kubernetes.io/docs/tutorials/) でチェックしたり、[kind](https://kind.sigs.k8s.io/)によるDockerコンテナーの "ノード" でKubernetesクラスターをローカルで実行してください。
[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)を使って1.24を簡単にインストールすることも可能です。

### Release Team

このリリースは、Kubernetes 1.24リリースチームを構成する献身的な個人の共同の努力なくしては不可能でした。このチームは、各Kubernetesリリースに含まれるコード、ドキュメント、リリースノートなどを含む全てのコンポーネントを提供するために集まりました。

リリースサイクルを成功させるために私たちを導いてくれたリリースリーダーのJamesLaverackに特に感謝します。 そして、Kubernetesコミュニティにv1.24リリースを提供するために、時間と労力を費やしてくれたすべてのリリースチームメンバーに感謝します。

### Release Theme and Logo

**Kubernetes 1.24: Stargazer**

![release-logo](https://kubernetes.io/images/blog/2022-05-03-kubernetes-release-1.24/kubernetes-1.24.png)

Kubernetes 1.24のテーマは _星を見つめる人(スターゲイザー)_ です。

古代の天文学者からJames Webb Space Telescopeを作り上げた科学者に至るまで、何世代もの人類は畏敬の念と驚きとともに星を見つめてきました。星は我々にインスピレーションを与え、想像力を育み、困難な航海の長い夜を案内してくれました。

このリリースで我々は、コミュニティが集まったときに何が可能になるか上を見つめて考えました。Kubernetesは世界中の何百ものコントリビュータと、何百万ものアプリケーションを提供する何千ものエンドユーザの作業の結晶です。

リリースロゴは[Britnee Laverack](https://www.instagram.com/artsyfie/)によって作られ、星がきらめく空と、しばしば "7人の姉妹" の神話として知られる[プレアデス](https://en.wikipedia.org/wiki/Pleiades)を覗く望遠鏡を描いています。7という数字は特にKubernetesプロジェクトにおいては特別で縁起の良いものです。そしてそれは元の "プロジェクトセブン" という名前にさかのぼるものです。

このKubernetesリリースは夜空を見つめ思いを巡らす人、つまり全ての星を見つめる人にちなんで名付けられました。✨

### User Highlights

* リテールEコマース企業の [La Redoute が Kubernetes と他のCNCFプロジェクトを使い、開発から運用までのソフトウェアデリバリのライフサイクルの変更と簡素化](https://www.cncf.io/case-studies/la-redoute/) をどのようにしてリードしたかをチェックしてください。
* API呼び出しの変更が問題を起こさないようにするため、 [Salt Security はgRPCで通信しLinkerdでメッセージ暗号化されたマイクロサービスの全体をKubernetes上に構築しました。](https://www.cncf.io/case-studies/salt-security/)
* プライベートからパブリッククラウドへの移行を通して, [Allainz Direct のエンジニアはわずか3ヶ月でCI/CDパイプラインを再設計し、200のワークフローを10〜15まで縮めることに成功しました。](https://www.cncf.io/case-studies/allianz/)
* どのように[UKのフィンテック企業であるBinkが社内KubernetesディストリビューションをLinkerdとともにアップデートし、パフォーマンスや安定性を注意深く見ながら必要な時にスケールできるクラウドに依存しない(cloud-agnostic)なプラットフォームを構築したか](https://www.cncf.io/case-studies/bink/)をチェックしてください。
* Kubernetesを使うことで、 オランダの組織である [Stichting Open Nederland](http://www.stichtingopennederland.nl/) が1ヶ月半でオランダでの安全なイベントの再開を助けるための検査用ポータルを作りました。 [Testing for Entry (Testen voor Toegang)](https://www.testenvoortoegang.org/) プラットフォームは [Kuberntesの性能とスケーラビリティを用い、1日に40万人を超えるCOVID-19検査を予約できるようにしました。](https://www.cncf.io/case-studies/true/)
* SparkFabrik と Backstage を活用し、 [Santagostino は開発者用プラットフォームのSamaritanを作りました。それはサービスとドキュメントを一元化し、サービスのライフサイクル全体を管理し、Santagostino開発者の作業を簡素化します。](https://www.cncf.io/case-studies/santagostino/)

### Ecosystem Updates

* KubeCon + CloudNativeCon Europe 2022 がスペインのバレンシアで2022/05/16〜20で開催されます! [イベントサイト](https://events.linuxfoundation.org/archive/2021/kubecon-cloudnativecon-europe/)でカンファレンスの詳細の確認と登録が可能です。
* [2021年の Cloud Native Survey](https://www.cncf.io/announcements/2022/02/10/cncf-sees-record-kubernetes-and-container-adoption-in-2021-cloud-native-survey/)を通して、 CNCFは記録的なKubernetesとコンテナの適用を見てきました。[サーベイの結果](https://www.cncf.io/reports/cncf-annual-survey-2021/)を確認してみてください。
* The [Linux Foundation](https://www.linuxfoundation.org/) と [The Cloud Native Computing Foundation](https://www.cncf.io/) (CNCF) は、新規の [Cloud Native Developer Bootcamp](https://training.linuxfoundation.org/training/cloudnativedev-bootcamp/?utm_source=lftraining&utm_medium=pr&utm_campaign=clouddevbc0322) を発表しました。これは、参加者にクラウドネイティブアプリケーションの設計、構築、配備の知識と技術を提供します。より詳細は[アナウンス](https://www.cncf.io/announcements/2022/03/15/new-cloud-native-developer-bootcamp-provides-a-clear-path-to-cloud-native-careers/)を確認してください。

### Project Velocity

[CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m) プロジェクトは、Kubernetesやそのサブプロジェクトのベロシティに関する興味深いデータの数字を集約します。ここには個人のコントリビュータからコントリビューションしている企業の数まで含まれており、このエコシステムを進化させる努力の深さと広さを示しています。

v1.24 リリースサイクルは[17週間](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.24) (January 10 to May 3)でした。[1029の企業](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.23.0%20-%20now&var-metric=contributions) と [1179の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.23.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes)によるコントリビューションがありました。

## Upcoming Release Webinar

2022/05/24 9:45am – 11am PT にKubernetes 1.24のリリースチームメンバーがこのリリースの主要な機能やアップグレード計画を助けるための非推奨、削除機能について学ぶために(リリースウェビナーに)参加します。
詳細情報と登録のためには、CNCF Online Programsサイトの[イベントページ](https://community.cncf.io/e/mck3kd/) を参照してください。

## Get Involved

Kubernetesに参加するための最も簡単な方法は、多くの[Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md)(SIGs)の中で自分の興味のあるものに参加することです。 
もしコミュニティと共有したいことがある場合は、週次の[コミュニティミーティング](https://github.com/kubernetes/community/tree/master/communication)に参加することも可能ですし、以下のチャンネルも活用してください。

* [Kubernetes Contributors](https://www.kubernetes.dev/)でコントリビューションについての詳細を見つけられます。 
* 最新の更新についてはTwitterの[@Kubernetesio](https://twitter.com/kubernetesio)をフォローしてください。 
* コミュニティでの相談は[Discuss](https://discuss.kubernetes.io/)に参加してください。 
* [Slack](http://slack.k8s.io/)のコミュニティに参加してください。 
* 質問(や質問の回答)を[Server Fault](https://serverfault.com/questions/tagged/kubernetes)に登録してください。
* あなたのKubernetesの[ストーリー](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)を共有してください。 
* [blog](https://kubernetes.io/blog/)でKubernetesで起きてる出来事を読むことが出来ます。
* [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)についての詳細を知ることが出来ます。


