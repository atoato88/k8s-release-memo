本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

原文:https://kubernetes.io/blog/2022/12/09/kubernetes-v1-26-release/

---

**Authors**: [Kubernetes 1.26 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.26/release-team.md)

Kubernetes v1.26 のリリースを発表できることを非常に嬉しく思います。

このリリースは合計で37個のEnhancementがあります。11個がStableとなり、10個がBetaとなり、16個がAlphaとなりました。また、12個の機能がDeprecatedあるいは削除されました。この内3個をこのアナウンスの中で詳細に述べます。

## Release theme and logo 

**Kubernetes 1.26: Electrifying**

![Kubernetes 1.26 Electrifying logo](https://d33wubrfki0l68.cloudfront.net/852be6960c90223d8d573098bc3f0c16b56b2252/b1ba9/images/blog/2022-12-08-kubernetes-1.26-release/kubernetes-1.26.png)

Kubernetes v1.26のテーマは _しびれること(Electrifying)_ です。

それぞれのKubernetesリリースは、献身的なボランティアによる強調した努力の結果です。
これは、世界中の複数のデータセンターと地域にある多様で複雑なコンピューティングリソースのセットを使用することで可能になりました。
リリースの最終成果物であるバイナリ、イメージコンテナ、ドキュメントは、ますます多くの個人利用、オンプレミス、およびクラウドコンピューティングリソースにデプロイされています。

今回のリリースでは、Kubernetesの開発および利用の基盤となるこれらすべての構成要素の重要性を認識すると同時に、Kubernetesを利用する際には、次の点に留意する必要があります。
環境持続可能性(Environmental Sustainability)は、あらゆるソフトウェアソリューションの作成者と利用者にとって避けられない関心事です。
また、Kubernetesのようなソフトウェアの環境フットプリント(Environmental Footprint)は、今後のリリースで重要な役割を果たすと信じています。

コミュニティとして、私たちは常に新しいリリースのプロセスを以前より良くするために努力しています。（例えば、今回のリリースで私たちは[機能拡張の追跡にGitHubのProjectsの使用を開始しました](https://github.com/orgs/kubernetes/projects/98/views/1))。[v1.24 "Stargazer"](/blog/2022/05/03/kubernetes-1-24-release-announcement/) では _もし私たちが上を向いて、私たちのコミュニティが一緒になれば何ができるかを考えていました。_ そして[v1.25 "Combiner"](/blog/2022/08/23/kubernetes-v1-25-release/)では、 私たちのコミュニティが力を合わせると何ができるのか？_ を考えていました。
このv1.26 "Electrifying"もまた、リリースフローに統合された個人の動きで、私たちのコミュニティができることに貢献したすべての人に捧げます。

## Major themes

Kubernetes v1.26は、世界中のボランティアチームによってもたらされた多くの変更で構成されています。今回のリリースでは、いくつかの主要なテーマがあります。

### Change in container image registry

前回のリリースで、[Kubernetesはコンテナレジストリ](https://github.com/kubernetes/kubernetes/pull/109938)を変更し、複数のクラウドプロバイダやリージョンに負荷を分散できるようにしました。1つのエンティティへの依存を削減し、多くのユーザーに対してより高速なダウンロード体験を提供するための変更でした。

今回のKubernetesのリリースは、はじめて`registry.k8s.io`に排他的に公開される最初のリリースです。(現在はレガシーな) `k8s.gcr.io` イメージレジストリでは、v1.26 用のコンテナイメージのタグは公開されません。v1.26 用のタグは発行されず、v1.26 より前のリリースのタグのみが引き続き更新されます。[registry.k8s.io:  faster, cheaper and GenerallyAvailable](/blog/2022/11/28/registry-k8s-io-faster-cheaper-ga/)を参照してください。
この重要な変更の動機、利点、および意味について、より詳しい情報を得ることができます。

### CRI v1alpha2 removed

[Container Runtime Interface](/docs/concepts/architecture/cri/) (CRI)の採用や[dockershimの削除](/blog/2022/02/17/dockershim-faq/) がv1.24で実施されたことにより、CRIは、Kubernetesが異なるコンテナと対話するために唯一サポートされ、かつ文書化された方法となりました。各kubeletは、そのノード上のコンテナランタイムと使用するCRIのバージョンをネゴシエートします。

以前のリリースでは、KubernetesプロジェクトはCRIバージョン`v1`の使用を推奨していました。kubeletはまだ CRI `v1alpha2` の使用をネゴシエートできましたが、これは非推奨となりました。

Kubernetes v1.26 では、CRI `v1alpha2` のサポートが打ち切られました。 その[削除](https://github.com/kubernetes/kubernetes/pull/110618) は、コンテナランタイムがCRI `v1` をサポートしていない場合、kubeletがノードを登録しない結果になります。
コンテナランタイムがCRI `v1` をサポートしていない場合、kubeletがノードを登録しないという結果になります。これは、containerdのマイナーバージョン1.5以前がKubernetes 1.26でサポートされないということです。containerd を使用する場合は、そのノードを Kubernetes にアップグレード **する前** に containerd をバージョン 1.6.0 またはそれ以降にアップグレードする必要があります。これは `v1alpha2` のみをサポートする他のコンテナランタイムにも同様に当てはまります。コンテナランタイムのベンダーに相談するか、そのウェブサイトをチェックしてください。

### Storage improvements

CSIの移行は、以前のリリースに含まれていた[コアのコンテナ・ストレージ・インターフェイス（CSI）マイグレーション](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/625-csi-migration)のGAに続き、これまで数回のリリースで取り組んできた継続的な取り組みです。このリリースでは、Kubernetesストレージの他の改善と同様に、移行の目標に沿った機能を追加（および削除）しています。

#### CSI migration for Azure File and vSphere graduated to stable

[vSphere](https://github.com/kubernetes/enhancements/issues/1491)と[Azure](https://github.com/kubernetes/enhancements/issues/1885)のインツリードライバのCSIへの移行はStableとなりました。これらに関する詳細な情報は、[vSphere CSI driver](https://github.com/kubernetes-sigs/vsphere-csi-driver) および [Azure File CSI driver](https://github.com/kubernetes-sigs/azurefile-csi-driver) リポジトリを参照してください。

#### _Delegate FSGroup to CSI Driver_ graduated to stable

この機能により、Kubernetesは[ボリュームのマウント時にPodの`fsGroup`をCSIドライバに供給し](https://github.com/kubernetes/enhancements/issues/2317)、ドライバがボリュームパーミッションを制御するためのマウントオプションを利用できるようにします。
以前は、kubeletは常にPodの`.spec.securityContext.fsGroupChangePolicy`フィールドに指定されたポリシーに従って、ボリューム内のファイルに `fsGroup` のオーナーシップとパーミッションの変更を適用していました。
このリリースから、CSIドライバーには、ボリュームのアタッチ時またはマウント時に `fsGroup` 設定を適用するオプションがあります。

#### In-tree GlusterFS driver removal

v1.25リリースですでに非推奨となっていた、インツリーのGlusterFSドライバは、このリリースで[削除](https://github.com/kubernetes/enhancements/issues/3446)されました。

#### In-tree OpenStack Cinder driver removal

このリリースでは、OpenStack用の非推奨のインツリーのストレージ機能 (`cinder` ボリュームタイプ) が削除されました。外部クラウドプロバイダーとCSIドライバは、代わりに https://github.com/kubernetes/cloud-provider-openstack のサイトから移行してください。詳細については、[Cinder in-tree から CSI ドライバへの移行](https://github.com/kubernetes/enhancements/issues/1489) を参照してください。

### Signing Kubernetes release artifacts graduates to beta

Kubernetes v1.24で導入された、[この機能](https://github.com/kubernetes/enhancements/issues/3031)は、Kubernetesのリリースプロセスのセキュリティを向上させる上で重要なマイルストーンとなります。すべてのリリースアーティファクトは[cosign](https://github.com/sigstore/cosign/)を使用してキーレスで署名され、バイナリアーティファクトとイメージの両方が[検証可能](https://kubernetes.io/docs/tasks/administer-cluster/verify-signed-artifacts/)です。

### Support for Windows privileged containers graduates to stable

特権コンテナのサポートにより、コンテナはホスト上で直接実行されるプロセスと同様のアクセス権で実行することができます。Windowsノードでこの機能をサポートする、[HostProcess
コンテナ](/docs/tasks/configure-pod-container/create-hostprocess-pod/)が、[stableとなりました。](https://github.com/kubernetes/enhancements/issues/1981)特権コンテナからホストリソース(ネットワークリソースを含む)にアクセスできるようになります。

### Improvements to Kubernetes metrics

このリリースでは、メトリクスに関していくつかの注目すべき改善がなされています。

#### Metrics framework extension graduates to alpha

メトリックスフレームワークの拡張機能[がアルファとなりました](https://github.com/kubernetes/enhancements/issues/3498)。
[Kubernetesコードベースのすべてのメトリックについてドキュメントが公開されました](/docs/reference/instrumentation/metrics/)。
この拡張は、2つの追加のメタデータフィールドを追加します。それは`Internal`と`Beta`で、メトリックの成熟度の異なる段階を表しています。

#### Component Health Service Level Indicators graduates to alpha

また、Kubernetesのメトリクスを消費する機能が改善され、[component health Service Level Indicators (SLIs)](/docs/reference/instrumentation/slis/)が[アルファとなりました。](https://github.com/kubernetes/kubernetes/pull/112884) `ComponentSLIs` 機能フラグを有効にすることで、追加のメトリクスエンドポイントが利用できるようになります。それによりサービスレベルオブジェクティブ(SLO)の計算を可能にする追加のメトリクスエンドポイントが存在するようになります。

#### Feature metrics are now available

Kubernetesの各コンポーネントに対して機能メトリック（Feature metrics）が利用できるようになり、`kubernetes_feature_enabled` をチェックすることで、[各アクティブなフィーチャーゲートが有効かどうかを追跡する](https://github.com/kubernetes/kubernetes/pull/112690)ことが可能になりました。

### Dynamic Resource Allocation graduates to alpha

[ダイナミック・リソースアロケーション](/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)、サードパーティの開発者がリソースのスケジューリングを行えるようにする[新機能](/docs/cepts/scheduling-eviction/dynamic-resource-allocation/) です。
リソースへのアクセスを要求するための制限された "countable" インターフェースに代わるものです。
(例: `nvidia.com/gpu: 2`) リソースへのアクセスを要求するための限定的なより永続ボリュームに類似した API を提供します。内部実装としては、[Container Device Interface](CDI)(https://github.com/container-orchestrated-devices/container-device-interface)を使用して、デバイスインジェクションを行います。
この機能は `DynamicResourceAllocation` フィーチャーゲートでブロックされます。（アルファのため、デフォルトではOffということを言っていると思われる。）

### CEL in Admission Control graduates to alpha

[この機能](https://github.com/kubernetes/enhancements/issues/3488) は [validating admission policies](/docs/reference/access-authn-authz/validating-admission-policy/) の`v1alpha1` API を導入します。
有効にすることで、[Common Expression Language](https://github.com/google/cel-spec)式による拡張可能なアドミッション制御を可能にします。現在ではカスタムポリシーは [admission webhooks](/docs/reference/access-authn-authz/extensible-admission-controllers/) を介して実施されています。
これは柔軟性がある一方で、プロセス内のポリシー適用と比較すると、いくつかの欠点があります。CELを使用するには`ValidatingAdmissionPolicy`フィーチャーゲートと APIの`admissionregistration.k8s.io/v1alpha1`を`--runtime-config` 経由で有効にします。

### Pod scheduling improvements

Kubernetes v1.26では、スケジューリングの動作をより適切に制御できるよう、いくつかの関連する機能強化が導入されています。

#### `PodSchedulingReadiness` graduates to alpha

[この機能](https://github.com/kubernetes/enhancements/issues/3521)は、PodのAPIに `.spec.schedulingGates` フィールドを導入します。これによって[Podをスケジューリングから除外することを表現できます。](/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
外部ユーザー/コントローラーは、このフィールドを使用して、ポリシーとニーズに基づいてPodのスケジューリングを抑えることができます。

#### `NodeInclusionPolicyInPodTopologySpread` graduates to beta

`topologySpreadConstraints` で `nodeInclusionPolicy` を指定することで、以下のことが制御可能になります。
Pod Topology Spreadのスキューを計算するときに[TaintやTolerationを考慮](/docs/concepts/scheduling-eviction/topology-spread-constraints/) するかどうかを制御できます。

## Other Updates

### Graduations to stable

このリリースには、Stableに昇格した合計11の機能強化が含まれています。

* [Support for Windows privileged containers](https://github.com/kubernetes/enhancements/issues/1981)
* [vSphere in-tree to CSI driver migration](https://github.com/kubernetes/enhancements/issues/1491)
* [Allow Kubernetes to supply pod's fsgroup to CSI driver on mount](https://github.com/kubernetes/enhancements/issues/2317)
* [Azure file in-tree to CSI driver migration](https://github.com/kubernetes/enhancements/issues/1885)
* [Job tracking without lingering Pods](https://github.com/kubernetes/enhancements/issues/2307)
* [Service Internal Traffic Policy](https://github.com/kubernetes/enhancements/issues/2086)
* [Kubelet Credential Provider](https://github.com/kubernetes/enhancements/issues/2133)
* [Support of mixed protocols in Services with type=LoadBalancer](https://github.com/kubernetes/enhancements/issues/1435)
* [Reserve Service IP Ranges For Dynamic and Static IP Allocation](https://github.com/kubernetes/enhancements/issues/3070)
* [CPUManager](https://github.com/kubernetes/enhancements/issues/3570)
* [DeviceManager](https://github.com/kubernetes/enhancements/issues/3573)

### Deprecations and removals

このリリースで12個の機能が[非推奨または削除](/blog/2022/11/18/upcoming-changes-in-kubernetes-1-26/)され、Kubernetesから削除されました。
    
* [CRI `v1alpha2` API is removed](https://github.com/kubernetes/kubernetes/pull/110618)
* [Removal of the `v1beta1` flow control API group](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#flowcontrol-resources-v126)
* [Removal of the `v2beta2` HorizontalPodAutoscaler API](https://kubernetes.io/docs/reference/using-api/deprecation-guide/#horizontalpodautoscaler-v126)
* [GlusterFS plugin removed from available in-tree drivers](https://github.com/kubernetes/enhancements/issues/3446)
* [Removal of legacy command line arguments relating to logging](https://github.com/kubernetes/kubernetes/pull/112120)
* [Removal of `kube-proxy` userspace modes](https://github.com/kubernetes/kubernetes/pull/112133)
* [Removal of in-tree credential management code](https://github.com/kubernetes/kubernetes/pull/112341)
* [The in-tree OpenStack cloud provider is removed](https://github.com/kubernetes/enhancements/issues/1489)
* [Removal of dynamic kubelet configuration](https://github.com/kubernetes/kubernetes/pull/112643)
* [Deprecation of non-inclusive `kubectl` flag](https://github.com/kubernetes/kubernetes/pull/113116)
* [Deprecations for `kube-apiserver` command line arguments](https://github.com/kubernetes/kubernetes/pull/38186)
* [Deprecations for `kubectl run` command line arguments](https://github.com/kubernetes/kubernetes/pull/112261)

### Release notes

Kubernetes v1.26リリースの完全な詳細は、我々の[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.26.md)をご覧ください。

### Availability

Kubernetes v1.26は、[Kubernetesサイト](https://k8s.io/releases/download/)でダウンロードできます。
Kubernetesを使い始めるには、以下の[インタラクティブチュートリアル](/docs/tutorials/)をチェックアウトするか、またはローカルでコンテナを「ノード」として使用するKubernetesクラスタを[kind](https://kind.sigs.k8s.io/)で実行することもできます。
また [kubeadm](/docs/setup/independent/create-cluster-kubeadm/) を使って簡単にv1.26をインストールすることができます。

### Release team

Kubernetesは、そのコミュニティのサポート、コミットメント、そしてハードワークによってのみ実現可能です。各リリースチームは、献身的なコミュニティのボランティアで構成され、皆さんが信頼するKubernetesのリリースを構成する多くの部品を一緒に構築しています。
これには、コードや文書化、プロジェクト管理まで、私たちのコミュニティの隅々にいる人々の専門的なスキルを必要とします。

私たちは、コミュニティにKubernetes v1.26リリースを確実に提供するために多大な時間を費やしてくれた[リリースチーム](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.26/release-team.md)全体に感謝します。

リリース・リードのLeonard Pahlkeには、リリース・サイクル全体を通して、私たちリリース・チーム全体をうまく舵取りしてくれたことに、特別な感謝を捧げたいと思います。
このリリースで私たち全員が最高の形で貢献できるよう、リリースを成功に導くための多くの様々なな事柄に常にサポートと配慮を怠りませんでした。

### User highlights
    
* Wortellは、開発者の専門知識と日々のインフラ管理に要する時間がますます増えていくことに直面していました。[Daprを使用することで、インフラ関連のコードの複雑さと必要な量を削減し、より多くの時間を新しいインフラ管理に集中できるようになりました。](https://www.cncf.io/case-studies/wortell/)
* Utmostは、機密性の高い個人情報を取り扱うため、SOC 2 Type II認証、ISO 27001認証、ゼロ・トラストネットワーク認証が必要でした。[Ciliumを使用して、開発者が新しいポリシーを作成し、毎秒4,000以上のフローをサポートすることを可能にする自動パイプラインを作成しました。](https://www.cncf.io/case-studies/utmost/)
* グローバルなサイバーセキュリティ企業であるEricom社のソリューションは、超低遅延とデータセキュリティに依存しています。[RidgeのマネージドKubernetesサービスにより、彼らは単一のAPIを通じて、世界中のサービスプロバイダのネットワークにデプロイすることができました。](https://www.cncf.io/case-studies/ericom/)
* スカンジナビアのオンライン銀行である Lunarは、ディザスタリカバリに備え、四半期ごとに本番クラスタのフェイルオーバーテストを実施したいと考えていました。また、プラットフォームサービスを管理するためのより良い方法を必要としていました。
  [クラスターを接続するためにLinkerdを使用することで、ログ管理システムの一元化からスタートし、すべてのプラットフォームサービスを一元化しました。](https://www.cncf.io/case-studies/lunar/)
* Datadogは、複数のクラウドプロバイダーで10,000以上のノードと100,000以上のポッドを持つ数十のクラスタを実行しています。[CiliumをCNIとして使い kube-proxyの代替とし、eBPF のパワーを活用し、あらゆるクラウドでユーザーに一貫したネットワーク体験を提供することにしました。](https://www.cncf.io/case-studies/datadog/)
* Insielは、ソフトウェア制作の方法を更新し、クラウドネイティブパラダイムを導入することを望んでいました。[KiratechとMicrosoft Azureを使ったデジタルトランスフォーメーションプロジェクトにより、クラウドファーストの文化を発展させることができました。](https://www.cncf.io/case-studies/insiel/)

### Ecosystem updates

* KubeCon + CloudNativeCon Europe 2023 は、オランダのアムステルダムで 2023年4月17日 〜 21 日に開催されます。カンファレンスに関する詳細や参加登録は、[イベント
  サイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)をご覧ください。
* CloudNativeSecurityCon North Americaは、クラウドネイティブセキュリティプロジェクトのコラボレーション、ディスカッション、知識の共有し、セキュリティの課題と機会に対処するためにこれらをどのように活用するかを目的とした2日間のイベントです。アメリカのシアトルで2023年2月1日〜2日に開催されます。詳細は、[イベントページ](https://events.linuxfoundation.org/cloudnativesecuritycon-north-america/)をご覧ください。
* CNCFは【2022年コミュニティアワード受賞者】(https://www.cncf.io/announcements/2022/10/28/cloud-native-computing-foundation-reveals-2022-community-awards-winners/)を発表しました。コミュニティアワードは、クラウドネイティブテクノロジーを発展させ、さらにそれ以上のことをしているCNCFコミュニティメンバーを表彰するものです。

### Project velocity

[CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m)プロジェクトは、Kubernetesと様々なサブプロジェクトのベロシティに関連する多くの興味深いデータポイントを集約しています。これには、個人の貢献から、貢献している企業の数まで、あらゆるものが含まれています。このエコシステムを進化させるために費やされる努力の深さと広さを示しています。

v1.26 のリリースサイクルでは、[14 週間が費やされ](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.26)(9月5日から12月9日まで)、[976社](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.25.0%20-%20v1.26.0&var-metric=contributions) と [6877人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.25.0%20-%20v1.26.0&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes) からの貢献がありました。

## Upcoming Release Webinar

Join members of the Kubernetes v1.26 release team on Tuesday January 17, 2023 10am - 11am EST (3pm - 4pm UTC) to learn about the major features
of this release, as well as deprecations and removals to help plan for upgrades.  For more information and registration, visit the [event
page](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-v126-release/).

Kubernetes v1.26 リリースチームのメンバーが、2023年1月17日（火）10am 〜 11am EST (3pm - 4pm UTC) に、このリリースの主要機能についての情報を提供します。
このリリースの主な機能、およびアップグレードの計画に役立つ非推奨と削除について学べます。 
詳細と参加登録は、[イベントページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-v126-release/)をご覧ください。

## Get Involved

Kubernetesに参加する最も簡単な方法は、あなたの興味に合った数多くの[Special Interest Group](https://github.com/kubernetes/community/blob/master/sig-list.md)(SIGs)のいずれかに参加することです。

Kubernetes コミュニティに発信したいことがありますか？毎週開催している[コミュニティ・ミーティング](https://github.com/kubernetes/community/tree/master/communication)、および以下のチャネルで、あなたの声を共有してください。
* Kubernetesへのコントリビューションについては、[Kubernetes Contributors](https://www.kubernetes.dev/)のウェブサイトをご覧ください。
* 最新情報はTwitter[@Kubernetesio](https://twitter.com/kubernetesio)でフォローしてください。
* [Discuss](https://discuss.kubernetes.io/)のコミュニティディスカッションに参加してください。
* [Slack](http://slack.k8s.io/)のコミュニティに参加してください。
* [Server Fault](https://serverfault.com/questions/tagged/kubernetes)に質問を投稿してください。(または質問に答えてください。)
* あなたのKubernetesストーリーを[共有](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)してください。
* Kubernetesで何が起こっているかについては、[ブログ](https://kubernetes.io/blog/)をお読みください。
* [Kubernetesリリース](https://github.com/kubernetes/sig-release/tree/master/release-team)の詳細については、こちらをご覧ください。
