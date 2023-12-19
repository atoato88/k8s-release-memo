本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

原文: https://kubernetes.io/blog/2023/08/15/kubernetes-v1-28-release/

**Authors**: [Kubernetes v1.28 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.28/release-team.md)

Kubernetes v1.28 Planternetesのリリースを発表、2023年の第2弾です！

このリリースには45の機能拡張で構成されています。このうち19の機能拡張がAlphaに、14がBetaに、そして12がStableに移行しています。

## Release Theme And Logo

**Kubernetes v1.28: _Planternetes_**

Kubernetes v1.28のテーマは *Planternetes* です。

![Kubernetes 1.28 Planternetes logo](https://kubernetes.io/images/blog/2023-08-15-kubernetes-1.28-blog/kubernetes-1.28.png)

Kubernetesの各リリースは、私たちのコミュニティの何千人もの個人のハードワークの集大成です。このリリースを支えているのは、業界のベテラン、親、学生、オープンソース初心者など、さまざまなバックグラウンドを持つ人々です。私たちは、それぞれのユニークな経験を組み合わせて、世界的な影響力を持つ集合的な成果物を作り上げました。

庭のように、私たちのリリースには常に変化する成長、挑戦、機会があります。このテーマは、リリースの今日に至るまでの細心の注意、意図、努力を称えるものです。調和しながら、私たちはより良く成長していきます。

# What's New (Major Themes)

## Changes to supported skew between control plane and node versions

Kubernetes v1.28では、コアノードとコントロールプレーンのコンポーネント間でサポートされるスキュー(バージョン間のずれ)がマイナーバージョンで1つ分拡張され、 _n-2_ から _n-3_ に拡張されました。これにより、サポートされる最も古いマイナーバージョンのノードコンポーネント（kubeletおよびkube-proxy）が、サポートされる最も新しいマイナーバージョンのコントロールプレーンコンポーネント（kube-apiserver、kube-scheduler、kube-controller-manager、cloud-controller-manager）で動作するようになります。

ノードはワークロードが実行される場所であるため、クラスタ運用者の中にはノードのメンテナンス、特にノードの動作の変更を避ける人もいます。なぜならノードはワークロードが実行される場所だからです。kubeletのマイナーバージョンアップグレードの場合、サポートされるプロセスにはそのノードのDrain処理が含まれ、そのためそこで実行されていたPodが中断されます。非常に長い時間実行されるワークロードがあり、可能な限りPodを実行し続ける必要があるKubernetesのエンドユーザーにとって、ノードメンテナンスのために失われる時間を減らすことはメリットとなります。

Kubernetesの年間サポート期間では、すでに毎年のアップグレードが可能になっています。ユーザーは、最新のパッチバージョンにアップグレードしてセキュリティフィックスを取得し、1年に1度、3回のマイナーバージョンアップグレードを順次行って、サポートされる最新のマイナーバージョンに「追いつく」ことができます。

以前は、サポートされるスキュー(バージョン間のずれ)の範囲内に収まるように、クラスターオペレータは年に一度のアップグレードを計画していたとき、ノードを2回（おそらく数時間間隔）アップグレードする必要があったでしょう。しかし今は違います。 Kubernetes v1.28では、毎年1回だけノードのマイナーバージョンアップを行ってアップストリームのサポートを受けられるオプションがあります。
※[訳者注] 詳細は[KEPの図](https://github.com/kubernetes/enhancements/tree/master/keps/sig-architecture/3935-oldest-node-newest-control-plane#motivation)を参照。

クラスターをより頻繁にアップグレードし、最新の状態に保ちたいのであれば、それは問題ありません。それでも構いませんし、完全にサポートされています。

## Generally available: recovery from non-graceful node shutdown

Kubernetesでは、ノードが予期せずシャットダウンしたり、回復不可能な状態（おそらくハードウェア障害やOSの無応答が原因）に陥ったりした場合、その後にクリーンアップを行い、ステートフルなワークロードを別のノードで再起動できるようにすることができます。Kubernetes v1.28では、これが安定機能(stable)になりました。

これにより、元のノードがシャットダウンされたり、ハードウェア障害や壊れたOSなどの回復不可能な状態に陥った後でも、ステートフルなワークロードが別のノードに正常にフェイルオーバーできるようになります。

Kubernetesのv1.20より前のバージョンでは、Linux上でのノードシャットダウンの処理が欠けていました。kubeletはsystemdと統合し、グレースフル・ノードシャットダウンを実装しています（ベータで、デフォルトで有効になっています）。しかし、意図的なシャットダウンでもうまく処理されないことがあります：

- ノードが Windows を実行している
- ノードが Linux を実行しているが、(`systemd` ではなく) 別の `init` を使用している場合。
- シャットダウンがシステムの inhibitor lock メカニズムをトリガしない場合
- ノードレベルの設定エラー
  (`shutdownGracePeriod`と`shutdownGracePeriodCriticalPods`に適切な値を設定していないなど)。

ノードがシャットダウンまたは故障し、そのシャットダウンがkubeletによって検出されなかった場合、StatefulSetの一部であるPodは、シャットダウンしたノード上のterminatingステータスでスタックします。停止したノードが再起動すると、そのノードのkubeletは、Kubernetes APIがそのノードにバインドされているとまだ認識しているPodをクリーンアップ（`DELETE`）できます。
しかし、ノードが停止したままの場合、または再起動後にkubeletが起動できない場合、Kubernetesは代替Podを作成できない可能性があります。シャットダウンされたノードのkubeletが古いPodを削除できない場合、関連するStatefulSetは新しいPodを作成できません（同じ名前を持ってしまうからです）。

ストレージにも問題があります。Podで使用されているボリュームがある場合、既存の VolumeAttachments は（現在はシャットダウンされている状態の）元のノードから切り離されないため、これらのPodで使用されている PersistentVolumes を別の健全なノードにアタッチすることはできません。その結果、影響を受けたStatefulSet上で実行されているアプリケーションが正しく機能しなくなる可能性があります。シャットダウンされた元のノードが起動した場合、それらのPodはそのkubeletによって削除され、別の実行中のノードで新しいポッドを作成できます。
元のノードが立ち上がらない場合（[immutable infrastructure](https://glossary.cncf.io/immutable-infrastructure/)設計ではよくあることです）、それらのPodはシャットダウンしたノードの`Terminating`ステータスで永久に動けなくなります。


Non-graceful node shutdownの後にクリーンアップをトリガする方法の詳細については、[non-graceful node shutdown](/docs/concepts/architecture/nodes/#non-graceful-node-shutdown)を参照してください。
    
## Improvements to CustomResourceDefinition validation rules 

[Common Expression Language (CEL)](https://github.com/google/cel-go)は、[カスタムリソース](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)を検証するために使用できます。主な目的は、かつてはカスタムリソース定義(CRD)の作成者として、Webhookを設計して実装する必要があった検証ユースケースの大部分を（CELによって）可能にすることです。代わりに、ベータ機能として、CRDのスキーマに _validation式_ を直接追加することができます。

CRD は、自明でない検証を直接サポートする必要があります。Admission WebhookはCRDの検証をサポートしますが、CRDの開発と運用を著しく複雑にします。

1.28では、2つのオプションフィールド `reason` と `fieldPath` が追加され、バリデーションに失敗したときに失敗理由とフィールドパスを指定できるようになりました。

詳細については、CRDドキュメントの[validation rules](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#validation-rules)を参照してください。

## ValidatingAdmissionPolicies graduate to beta

アドミッション制御のためのCommon Expression Languageは、Admission Webhookで検証する代わりに、Kubernetes APIサーバーへのリクエストをプロセス内で検証するカスタマイズ可能なものです。

これは、1.25でベータ版に移行したCRD Validation Rules機能の機能をベースにしていますが、Admission Controlの検証というポリシー実施機能に重点を置いています。

これにより、カスタマイズ可能なポリシーを実施するためのインフラの障壁が低くなるとともに、コミュニティがk8sとその拡張機能の両方のベストプラクティスを確立し、遵守するのに役立つプリミティブが提供されます。

[ValidatingAdmissionPolicies](/docs/reference/access-authn-authz/validating-admission-policy/) を使用するには、クラスターのコントロールプレーンで `admissionregistration.k8s.io/v1beta1` API グループと `ValidatingAdmissionPolicy` フィーチャーゲートの両方を有効にする必要があります。

## Match conditions for admission webhooks

Kubernetes v1.27では、Admission Webhookに _match conditions_ を指定できるようになり、Admission時にKubernetesがリモートHTTPコールを行う際のスコープを狭めることができます。
ValidatingWebhookConfigurationとMutatingWebhookConfigurationの`matchCondition`フィールドは、AdmissionのリクエストをWebhookに送信するためにtrueと評価されなければならないCEL式です。

Kubernetes v1.28では、このフィールドはベータ版に移行し、デフォルトで有効になっています。
    
詳しくはKubernetesドキュメントの[`matchConditions`](/docs/reference/access-authn-authz/extensible-admission-controllers/#matching-requests-matchconditions)を参照してください。

## Beta support for enabling swap space on Linux
    
これは、Kubernetesユーザーがテストを実行し、swapの上にクラスタ機能を構築し続けるためのデータを提供できるように、制御された予測可能な方法でノードにswapサポートを追加します。

スワップには2つの異なるタイプのユーザーがいますが、重複する可能性があります。

- ノード管理者は、ノードレベルのパフォーマンスチューニングや安定性/ノイジーネイバー問題の軽減のためにスワップを利用したい場合があります。

- アプリケーション開発者は、スワップ・メモリを使用することで恩恵を受けるアプリケーションを開発します。
    
## Mixed version proxy (alpha) {#mixed-version-proxy}

クラスタに複数のAPIサーバがあり、バージョンが混在している場合（アップグレード/ダウングレード時や、ランタイムコンフィグが変更されロールアウトが発生した場合など）、すべてのapiserverがすべてのバージョンですべてのリソースに対応できるわけではありません。

Kubernetes v1.28では、APIサーバーのアグリゲーションレイヤー内で _mixed version proxy_ を有効にすることができます。
混合バージョンのプロキシは、ローカルAPIサーバーは認識しないが、コントロールプレーン内の別のAPIサーバーはサポートできるリクエストを見つけます。適切なピアを見つけると、アグリゲーションレイヤーはリクエストを互換性のあるAPIサーバーにプロキシします。
    
クラスタ上でアップグレードまたはダウングレードが実行されると、しばらくの間コントロールプレーン内のAPIサーバのバージョンが異なることがあります。この新しいアルファ機構により、クライアントからそのズレ(Skew)を隠すことができます。
  
## Source code reorganization for control plane components

Kubernetesのコントリビューターは、[k/apiserver](https://github.com/kubernetes/apiserver)を消費しながらも、再利用可能なkube-apiserverの機能のより大きな、慎重に選択されたサブセットを持つ新しいステージングリポジトリ上に構築するために、kube-apiserverのコードの再編成を開始しました。

これは段階的な再編成です。最終的には、KubernetesのAPIサーバーから抽象化された汎用的な機能を持つ新しいgitリポジトリが存在することになるでしょう。

## Support for CDI injection into containers (alpha) {#cdi-device-plugin}

CDI は、複雑なデバイスをコンテナに注入するための標準化された方法を提供します（つまり、論理的に動作するために注入される /dev ノードが 1 つだけでないデバイス）。この新機能により、プラグイン開発者は、1.27でCRIに追加されたCDIDevicesフィールドを利用して、CDIデバイスをCDI対応ランタイム（最近のリリースではcontainerdとcrio-o）に直接渡すことができます。
    
## API awareness of sidecar containers (alpha) {#sidecar-init-containers}

Kubernetes 1.28では、[initコンテナ](https://github.com/kubernetes/website/blob/main/content/en/docs/concepts/workloads/pods/init-containers.md)にアルファの`restartPolicy`フィールドが導入され、initコンテナが _sidecarコンテナ_ でもあることを示すために使用されます。
kubelet は init コンテナを `restartPolicy： Always`を持つinitコンテナを、他のinitコンテナとともに定義された順番に起動します。
サイドカーコンテナの完了を待ってからPodのメインコンテナを起動するのではなく、kubeletはサイドカーinitコンテナの起動のみを待ちます。

Startupプローブが成功し、postStart ハンドラーが完了すると、kubelet はサイドカー コンテナの起動が完了したとみなします。
この条件は、ContainerStatus 型のフィールド Started で表されます。
起動プローブを定義しない場合、kubelet は postStart ハンドラの完了直後にコンテナの起動が完了したとみなします。

init コンテナでは、`restartPolicy` フィールドを省略するか、`Always` に設定します。このフィールドを省略することは、アプリケーションの起動前に完了まで実行される真の init コンテナが欲しいことを意味します。

サイドカーコンテナは Pod の完了をブロックしません。通常のコンテナがすべて完了すると、その Pod 内のサイドカーコンテナは終了します。

サイドカーコンテナが起動し（プロセスが実行され、`postStart` が成功し、設定された起動プローブが通過し）、障害が発生すると、Pod 全体の `restartPolicy` が `Never` または `OnFailure` であってもサイドカーコンテナは再起動されます。
さらに、サイドカーコンテナは (失敗時または正常終了時に) _Pod の終了中でも_ 再起動されます。

詳しくは [API for sidecar containers](/docs/concepts/workloads/pods/init-containers/#api-for-sidecar-containers) を参照してください。

## Automatic, retroactive assignment of a default StorageClass graduates to stable

Kubernetesは、PersistentVolumeClaim（PVC）に値を指定しない場合、自動的に`storageClassName`を設定します。コントロールプレーンは、`storageClassName`が定義されていない既存のPVCに対してもStorageClassを設定します。
以前のバージョンのKubernetesでもこの動作がありました。Kubernetes v1.28では自動で常にアクティブになります。この機能は安定版（一般提供）になりました。

詳細については、Kubernetesドキュメントの[StorageClass](/docs/concepts/storage/storage-classes/)を参照してください。
             
## Pod replacement policy for Jobs (alpha) {#pod-replacement-policy}

Kubernetes 1.28では、ジョブAPIに新しいフィールドが追加され、前のPodが終了し始めるとすぐに新しいPodを作成する（既存の動作）か、既存のPodが完全に終了してから新しいPodを作成する（新しいオプションの動作）かを指定できるようになりました。
    
TensorflowやJAXなどの一般的な機械学習フレームワークの多くは、インデックスごとに一意のPodを必要とします。
旧来の動作では、`Indexed` ジョブに属するPodが（先取り、立ち退き、その他の外的要因によって）終了状態になると、代替のPodが作成されますが、まだシャットダウンしていない古いPodと衝突してすぐに起動できなくなります。

前のPodが完全に終了する前に交換用のPodが現れると、リソースが不足しているクラスタや予算が厳しいクラスタでも問題が発生する可能性があります。これらのリソースの入手が困難なため、既存のPodが終了してからでないとPodがノードを見つけられないことがあります。クラスタの自動スケーラが有効になっている場合、交換用のPodを早期に作成すると、望ましくないスケールアップが発生する可能性があります。

詳しくは、ジョブドキュメントの [Delayed creation of replacement pods](/docs/concepts/workloads/controllers/job/#delayed-creation-of-replacement-pods) を読んでください。
  
## Job retry backoff limit, per index (alpha) {#job-per-index-retry-backoff}

これはジョブ API を拡張して、がインデックス単位のバックオフリミットをインデックス付きジョブでサポートし、いくつかのインデックスが失敗してもジョブの実行を継続できるようにします。
    
現在、インデックス付きジョブのインデックスは単一のバックオフリミットを共有しています。ジョブがこの共有バックオフリミットに達すると、ジョブコントローラはジョブ全体を失敗としてマークし、まだ完了まで実行されていないインデックスを含むリソースがクリーンアップされます。

その結果、既存の実装では、作業負荷が真に[embarrassingly parallel](https://en.wikipedia.org/wiki/Embarrassingly_parallel)であり、各インデックスが他のインデックスから完全に独立している状況をカバーできませんでした。

たとえば、インデックス化されたジョブを長時間実行される統合テスト群の基盤として使用する場合、各テストの実行は単一のテストの失敗を見つけることしかできないでしょう。

詳細については、Kubernetesドキュメントの[Handling Pod and container failures](/docs/concepts/workloads/controllers/job/#handling-pod-and-container-failures)を読んでください。

<hr />
<a id="cri-container-and-pod-statistics-without-cadvisor" />

**訂正**: cAdvisorを使用しないCRIコンテナとポッドの統計機能はリリースに間に合わなかったため削除しました。
最初のリリースアナウンスでは、Kubernetes 1.28にこの新機能が含まれていると記載されていました。

## Feature graduations and deprecations in Kubernetes v1.28
### Graduations to stable

このリリースには、Stable に昇格した合計12の機能強化が含まれています。

* [`kubectl events`](https://github.com/kubernetes/enhancements/issues/1440)
* [Retroactive default StorageClass assignment](https://github.com/kubernetes/enhancements/issues/3333)
* [Non-graceful node shutdown](https://github.com/kubernetes/enhancements/issues/2268)
* [Support 3rd party device monitoring plugins](https://github.com/kubernetes/enhancements/issues/606)
* [Auth API to get self-user attributes](https://github.com/kubernetes/enhancements/issues/3325)
* [Proxy Terminating Endpoints](https://github.com/kubernetes/enhancements/issues/1669)
* [Expanded DNS Configuration](https://github.com/kubernetes/enhancements/issues/2595)
* [Cleaning up IPTables Chain Ownership](https://github.com/kubernetes/enhancements/issues/3178)
* [Minimizing iptables-restore input size](https://github.com/kubernetes/enhancements/issues/3453)
* [Graduate the kubelet pod resources endpoint to GA](https://github.com/kubernetes/enhancements/issues/3743)
* [Extend podresources API to report allocatable resources](https://github.com/kubernetes/enhancements/issues/2403)
* [Move EndpointSlice Reconciler into Staging](https://github.com/kubernetes/enhancements/issues/3685)

### Deprecations and removals

削除

* [Removal of CSI Migration for GCE PD](https://github.com/kubernetes/enhancements/issues/1488)

非推奨

* [Ceph RBD in-tree plugin](https://github.com/kubernetes/kubernetes/pull/118303)
* [Ceph FS in-tree plugin](https://github.com/kubernetes/kubernetes/pull/118143)

## Release Notes

Kubernetes v1.28リリースの全詳細は[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.28.md)をご覧ください。

## Availability

Kubernetes v1.28は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.28.0)からダウンロード可能です。Kubernetesを使い始めるには、[minikube](https://minikube.sigs.k8s.io/docs/)や[kind](https://kind.sigs.k8s.io/)などを使用してローカルのKubernetesクラスタを実行できます。[kubeadm](/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)を使ってv1.28を簡単にインストールすることもできます。
 
## Release Team

Kubernetesは、そのコミュニティのサポート、コミットメント、そしてハードワークによってのみ実現可能です。各リリースチームは、献身的なコミュニティボランティアで構成され、皆さんが信頼するKubernetesのリリースを構成する多くのパーツを協力して構築しています。これには、コードそのものからドキュメント、プロジェクト管理に至るまで、コミュニティのあらゆる場所にいる人々の専門的なスキルが必要です。

私たちのコミュニティのためにKubernetes v1.28リリースを確実に提供するために懸命に仕事に費やした時間に対して、リリースチーム全員に感謝したいと思います。

スムーズで成功したリリース・サイクルを導いてくれたリリース・リードのGrace Nguyenに特に感謝します。

## Ecosystem Updates

* KubeCon + CloudNativeCon China 2023は、2023年9月26日～28日に中国の上海で開催されます！[イベントサイト](https://www.lfasiallc.com/kubecon-cloudnativecon-open-source-summit-china/)でカンファレンスと登録に関する詳細情報をご覧いただけます。
* KubeCon + CloudNativeCon North America 2023は、2023年11月6日～9日に米国イリノイ州シカゴで開催されます！[イベントサイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/)でカンファレンスと参加登録に関する詳細情報をご覧いただけます。

## Project Velocity

[CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m)プロジェクトは、Kubernetesと様々なサブプロジェクトの速度に関連する多くの興味深いデータを集約しています。これには、個人のコントリビューションからコントリビューションしている企業の数まであらゆるものが含まれ、このエコシステムを進化させるために費やされている努力の深さと広さを示しています。

[14週間](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.28)(5月15日から8月15日まで)実施されたv1.28リリースサイクルでは、[911社](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.27.0%20-%20now&var-metric=contributions)と[1440人の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.27.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-repo_name=kubernetes%2Fkubernetes&var-country_name=All&var-companies=All)からの貢献がありました。

## Upcoming Release Webinar

2023年9月6日（水）午前9時（PDT）より、Kubernetes v1.28リリースチームのメンバーと一緒に、このリリースの主要機能、およびアップグレードの計画に役立つ非推奨機能や削除機能について学びましょう。詳細とお申し込みは、CNCF Online Programs サイトの [イベントページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-128-release/) をご覧ください。

## Get Involved

Kubernetesに参加する最も簡単な方法は、あなたの興味に沿った多くの[Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs)の1つに参加することです。

Kubernetesコミュニティに発信したいことがありますか？私たちが毎週開催している[コミュニティミーティング](https://github.com/kubernetes/community/tree/master/communication)や以下のチャンネルであなたの声を共有してください。

* [Kubernetes Contributors website](https://www.kubernetes.dev/)で、Kubernetesへの貢献についての詳細をご覧ください。

* 最新のアップデートはTwitter [@Kubernetesio](https://twitter.com/kubernetesio)でフォローしてください。

* [Discuss](https://discuss.kubernetes.io/)でコミュニティの議論に参加してください。

* [Slack](https://communityinviter.com/apps/kubernetes/community) でコミュニティに参加してください。

* [Server Fault](https://serverfault.com/questions/tagged/kubernetes)に質問を投稿してください。

* あなたのKubernetesストーリーを[Share](https://docs.google.com/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)してください。

* [ブログ](https://kubernetes.io/blog/)でKubernetesで何が起こっているのか、もっと読んでください。

* [Kubernetesリリースチーム](https://github.com/kubernetes/sig-release/tree/master/release-team)についてもっと知ってください。
