本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

原文: https://kubernetes.io/blog/2024/04/17/kubernetes-v1-30-release/

**Editors:** Amit Dsouza, Frederick Kautz, Kristin Martin, Abigail McCarthy, Natali Vlatko

Kubernetes v1.30のリリースを発表：Uwubernetes、最もかわいいリリース！

Kubernetes v1.30のリリースでは、これまでのリリースと同様に、新しい安定版、ベータ版、アルファ版の機能が導入されています。
一流のリリースが一貫して提供されていることは、私たちの開発サイクルの強さと、コミュニティからの活発なサポートを裏付けています。

このリリースは45の機能強化で構成されています。これらの機能強化のうち、17項目が安定版（Stable）に、18項目がベータ版（Beta）に、10項目がアルファ版（Alpha）に移行しています。

## Release theme and logo

Kubernetes v1.30: *Uwubernetes*

![Kubernetes 1.29 Mandala logo](https://kubernetes.io/images/blog/2024-04-17-kubernetes-1.30-release/k8s-1.30.png)

Kubernetes v1.30はあなたのクラスタをよりかわいくする！

Kubernetesは、世界中、そしてあらゆる職業の何千人もの人々によって構築され、リリースされています。ほとんどの貢献者は報酬を得ているわけではなく、楽しむため、問題を解決するため、何かを学ぶため、あるいは単純にコミュニティを愛するためにKubernetesを構築しています。私たちの多くは、ここで家を見つけ、友人を見つけ、キャリアを築きました。リリースチームは、Kubernetesの継続的な成長の一端を担えることを光栄に思います。

Kubernetesを構築した人たち、リリースする人たち、そしてすべてのクラスタをオンラインに保つふわふわの人(furries)たちのために、私たちはKubernetes v1.30: Uwubernetesを紹介します。この名前は "kubernetes "と "UwU "の合成語で、幸せやかわいらしさを示すために使われる顔文字です。

私たちはここで喜びを見つけましたが、私たちはまた、このコミュニティを奇妙ですばらしく、歓迎に満ちたものにしてくれる助けとなる、外の生活からの喜びをもたらしてきました。私たちの作品を皆さんと共有できることをとても嬉しく思います。

UwU ♥️

## Improvements that graduated to stable in Kubernetes v1.30 {#graduations-to-stable}

_これは、v1.30リリース後に安定版となった改良点の一部です。_

### Robust VolumeManager reconstruction after kubelet restart ([SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage))

これはボリュームマネージャーのリファクタリングで、kubeletの起動時に既存のボリュームのマウント方法に関する追加情報をkubeletに入力できるようにします。これにより、一般的には kubelet の再起動後またはマシンの再起動後のボリュームのクリーンアップがより堅牢になります。

これは、ユーザーまたはクラスタ管理者に変更をもたらすものではありません。私たちは、何か問題が発生した場合に以前の動作にフォールバックできるように、機能プロセスとフィーチャーゲート `NewVolumeManagerReconstruction` を使用していました。機能が安定となった今、フィーチャーゲートはロックされ、無効にすることはできません。

### Prevent unauthorized volume mode conversion during volume restore ([SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage))

Kubernetes v1.30では、スナップショットをPersistentVolumeにリストアする際、コントロールプレーンは常にボリュームモードへの不正な変更を防止します。クラスタ管理者として、リストア時にこの種の変更を許可する必要がある場合は、適切なIDプリンシパル（たとえば、ストレージ統合を表すServiceAccounts）にパーミッションを付与する必要があります。

**注意**  
アップグレード前に必要なアクションがあります。 external-provisioner `v4.0.0` および external-snapshotter `v7.0.0` では、`prevent-volume-mode-conversion` 機能フラグがデフォルトで有効になっています。[external-provisioner 4.0.0](https://github.com/kubernetes-csi/external-provisioner/releases/tag/v4.0.0)および[external-snapshotter v7.0.0](https://github.com/kubernetes-csi/external-snapshotter/releases/tag/v7.0.0)の「緊急アップグレードに関する注意事項」に記載されている手順を実行しない限り、VolumeSnapshotからPVCを作成する際にボリュームモードの変更が拒否されます。


この機能の詳細については、[スナップショットのボリュームモードを変換する](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#convert-volume-mode)も参照してください。


### Pod Scheduling Readiness ([SIG Scheduling](https://github.com/kubernetes/community/tree/master/sig-scheduling))

Kubernetes v1.27でベータ版に昇格した _Pod scheduling readiness_ は、今回のリリースで安定版に昇格しました。

この安定版となった機能により、Kubernetesは、クラスタがまだそのPodを実際にノードにバインドするためのリソースをプロビジョニングしていないときに、定義済みのPodをスケジュールしようとするのを回避できるようになりました。Podのスケジュールを許可するかどうかのカスタムコントロールによって、クォータメカニズムやセキュリティコントロールなどを実装することもできます。

重要なのは、これらのPodをスケジューリング対象外としてマークすることで、スケジューラが現在クラスタにあるノードにスケジューリングできないPodやスケジューリングしようとしないPodを処理する手間を省くことができるということです。[cluster autoscaling](https://kubernetes.io/docs/concepts/cluster-administration/cluster-autoscaling/)をアクティブにしている場合、スケジューリングゲートを使うとスケジューラの負荷を減らせるだけでなく、コストも節約できます。スケジューリングゲートがなければ、オートスケーラは起動する必要のないノードを起動してしまうかもしれません。

Kubernetes v1.30では、Podの`.spec.schedulingGates`を指定（または削除）することで、Podがスケジューリングの対象となる準備が整うタイミングを制御できます。これは安定した機能であり、現在では正式にPodのKubernetes API定義の一部となっています。

### Min domains in PodTopologySpread ([SIG Scheduling](https://github.com/kubernetes/community/tree/master/sig-scheduling))

このリリースでは、PodTopologySpread制約の`minDomains`パラメータが安定版に卒業し、ドメインの最小数を定義できるようになりました。この機能はCluster Autoscalerで使用するように設計されています。

ドメインの使用を試みて、十分な数のドメインが存在しなかった場合、Podはスケジューリング不能としてマークされます。その後、Cluster Autoscalerが新しいドメインにノードをプロビジョニングし、最終的にPodが十分なドメインに広がるようになります。

### Go workspaces for k/k ([SIG Architecture](https://github.com/kubernetes/community/tree/master/sig-architecture))

KubernetesレポはGoワークスペースを使用するようになりました。これはエンドユーザーには全く影響しないはずですが、ダウンストリームのプロジェクト開発者には影響があります。ワークスペースに切り替えたことで、様々な[k8s.io/code-generator](https://github.com/kubernetes/code-generator)ツールのフラグがいくつか変更されました。ダウンストリームのコンシューマは [`staging/src/k8s.io/code-generator/kube_codegen.sh`](https://github.com/kubernetes/code-generator/blob/master/kube_codegen.sh) を見て変更を確認してください。

変更の詳細とGoワークスペースが導入された理由については、[Using Go workspaces in Kubernetes](https://www.kubernetes.dev/blog/2024/03/19/go-workspaces-in-kubernetes/)をお読みください。

## Improvements that graduated to beta in Kubernetes v1.30 {#graduations-to-beta}

_これは、v1.30リリース後にベータ版となった改良点の一部です。_

### Node log query ([SIG Windows](https://github.com/kubernetes/community/tree/master/sig-windows))

ノード上の問題のデバッグを支援するために、Kubernetes v1.27ではノード上で実行されているサービスのログを取得できる機能が導入されました。この機能を使用するには、そのノードで `NodeLogQuery` フィーチャーゲートが有効になっており、kubelet の設定オプション `enableSystemLogHandler` と `enableSystemLogQuery` が両方とも true に設定されていることを確認してください。

v1.30リリース後、これはベータ版になりました（ただし、この機能を使用するには、明示的に有効の設定にする必要があります）。

Linuxでは、サービスログがjournald経由で利用可能であることが前提です。Windowsでは、サービスログはアプリケーション・ログ・プロバイダで利用可能であることが前提です。ログは、`/var/log/` (Linux)または`C:¥varlog¥log` (Windows)内のファイルを読むことでも利用できます。詳細については、[log query](https://kubernetes.io/docs/concepts/cluster-administration/system-logs/#log-query) ドキュメントを参照してください。

### CRD validation ratcheting ([SIG API Machinery](https://github.com/kubernetes/community/tree/master/sig-api-machinery))

この動作を使用するには、`CRDValidationRatcheting` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)を有効にする必要があります。そうすることであなたのクラスターのCustomResourceDefinitionsに対して適用されます。

フィーチャーゲートを有効にすると、KubernetesはCustomResourceDefinitionsに対して _validation racheting_ を実装します。APIサーバーは、バリデーションに失敗したリソースの各部分が更新操作によって変更されていないことを条件に、更新後に有効でないリソースへの更新を受け入れます。言い換えると、無効なままのリソースの無効な部分は、すでに間違っている必要があります。このメカニズムを使用して、有効なリソースを無効となるように更新することはできません。

この機能により、CRDの作成者は、特定の条件下で自信を持ってOpenAPIV3スキーマに新しい検証を追加することができます。ユーザーは、オブジェクトのバージョンを上げたり、ワークフローを壊したりすることなく、安全に新しいスキーマに更新することができます。

※訳者注：有効なリソースはそのまま有効であることを担保できる機能が本機能であり、そのため安全にCRDの更新が行えるということのようです。

### Contextual logging ([SIG Instrumentation](https://github.com/kubernetes/community/tree/master/sig-instrumentation))

このリリースでは、Contextual Loggingがベータ版となり、開発者とオペレータが `WithValues` と `WithName` を通じて、サービス名やトランザクション ID のようなカスタマイズ可能で相関性のあるコンテキストの詳細をログに注入できるようになりました。この機能強化により、分散システム全体のログデータの相関と分析が簡素化され、トラブルシューティングの効率が大幅に向上します。Kubernetes環境の動作に対するより明確な洞察を提供することで、Contextual Loggingは運用上の課題をより管理しやすくし、Kubernetesの可観測性における顕著な進歩となります。

### Make Kubernetes aware of the LoadBalancer behaviour ([SIG Network](https://github.com/kubernetes/community/tree/master/sig-network))

`LoadBalancerIPMode` フィーチャーゲートがベータ版となり、デフォルトで有効になりました。この機能により、`type` が `LoadBalancer` に設定されたサービスの `.status.loadBalancer.ingress.ipMode` を設定することができます。`.status.loadBalancer.ingress.ipMode`はロードバランサーのIPがどのように動作するかを指定する。これは `.status.loadBalancer.ingress.ip` フィールドも指定されている場合にのみ指定できます。詳細は、[ロードバランサーステータスのIPMode指定](https://kubernetes.io/docs/concepts/services-networking/service/#load-balancer-ip-mode) を参照してください。

### Structured Authentication Configuration ([SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth))

_構造化認証設定(Structured Authentication Configuration)_ はこのリリースでベータ版を卒業しました。

Kubernetesには、より柔軟で拡張性の高い認証システムに対する長年のニーズがありました。現在のシステムは強力ではありますが、特定のシナリオで使用することを難しくするいくつかの制限があります。例えば、同じタイプの複数の認証子（例えば複数のJWT認証子）を使用したり、APIサーバーを再起動せずに設定を変更したりすることはできません。Structured Authentication Configuration 機能は、これらの制限に対処し、Kubernetesで認証を設定するための、より柔軟で拡張可能な方法を提供するための第一歩です。詳細は[Structured Authentication Configuration](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#using-authentication-configuration)を参照してください。

### Structured Authorization Configuration ([SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth))

_構造化認可設定(Structured Authorization Configuration)_ はこのリリースでベータ版を卒業しました。

Kubernetesは、システム管理者や開発者の複雑な要件を満たすために進化し続けています。クラスタのセキュリティと完全性を保証するKubernetesの重要な側面は、APIサーバーの認可です。最近まで、kube-apiserverの認可チェーンの設定はやや堅苦しく、コマンドラインフラグのセットに制限され、認可チェーンでは単一のWebhookしか許可されていませんでした。このアプローチは機能的ではあるものの、クラスタ管理者が複雑で、きめ細かな認証ポリシーを定義するために必要な柔軟性を制限していました。最新の[構造化認可設定]機能は、複数のWebhookを有効にし、明示的な制御メカニズムを提供することに重点を置き、認可チェーンをより構造化された汎用的な方法で設定することで、この点に革命を起こすことを目的としています。詳細は[Structured Authorization Configuration](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#configuring-the-api-server-using-an-authorization-config-file)を参照してください。

## New alpha features

### Speed up recursive SELinux label change ([SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage))

v1.27リリースから、Kubernetesにはすでに、ボリュームのコンテンツにSELinuxラベルを設定する最適化が含まれており、その機能は固定的なオーバーヘッドで動作します。Kubernetesは、マウントオプションを使用してこの高速化を実現しています。より遅いレガシーな動作では、コンテナランタイムがボリューム全体を再帰的に操作していき、各ファイルとディレクトリに個別にSELinuxラベルを適用する必要があるため、大量のファイルとディレクトリがあるボリュームでは特に処理の遅さが顕著になります。

Kubernetes v1.27で、この機能はベータ版として卒業しましたが、ReadWriteOncePodボリュームに限定していました。対応するフィーチャーゲートは `SELinuxMountReadWriteOncePod` です。デフォルトで有効になっており、v1.30でもベータ版のままです。

Kubernetes v1.30では、SELinuxマウントオプションのサポートがアルファ版として **すべての** ボリュームに拡張され、別のフィーチャーゲート `SELinuxMount` が追加されました。このフィーチャーゲートは、異なる SELinux ラベルを持つ複数の Pod が同じボリュームを共有する場合の動作の変更が発生します。詳細は[KEP](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/1710-selinux-relabeling#behavioral-changes)を参照してください。

SELinuxを有効にしてKubernetesを実行しているユーザーには、この機能をテストして[KEP issue](https://kep.k8s.io/1710)にフィードバックを提供することを強く推奨します。

| フィーチャーゲート|v1.30でのステージ|動作変更
|---|---|---| 
| SELinuxMountReadWriteOncePod |  Beta | No |
| SELinuxMount | Alpha | Yes | 

すべてのボリュームでこの機能をテストするには、`SELinuxMountReadWriteOncePod` と `SELinuxMount` の両方の機能ゲートを有効にする必要があります。

この機能はWindowsノードやSELinuxをサポートしていないLinuxノードでは効果がありません。

### Recursive Read-only (RRO) mounts ([SIG Node](https://github.com/kubernetes/community/tree/master/sig-node))

Alpha機能であるRRO（Recursive Read-Only）マウントの導入により、データのセキュリティに新たなレイヤーが加わりました。この機能により、ボリュームとそのサブマウントを読み取り専用に設定し、偶発的な変更を防ぐことができます。データの完全性が鍵となる重要なアプリケーションを導入することを想像してみてください。RROマウントは、データが変更されないことを保証し、クラスタのセキュリティをさらに強化します。これは、わずかな変更でも重大な影響を及ぼしかねない、厳重に管理された環境では特に重要です。

### Job success/completion policy ([SIG Apps](https://github.com/kubernetes/community/tree/master/sig-apps))

Kubernetes v1.30から、インデックス化されたジョブは `.spec.successPolicy` をサポートし、成功したPodに基づいてジョブが成功したと宣言できるタイミングを定義できるようになりました。これにより、2種類の基準を定義することができます。

- `succeededIndexes` は他のインデックスが失敗しても、これらのインデックスが成功したときにジョブが成功したと宣言できることを示します。
- `succeededCount`は、成功したインデックスの数がこの基準に達したときに、ジョブが成功したと宣言できることを示します。

ジョブが成功ポリシーを満たした後、ジョブコントローラは残っている Pod を終了させます。

### Traffic distribution for services ([SIG Network](https://github.com/kubernetes/community/tree/master/sig-network))

Kubernetes v1.30では、アルファ版としてKubernetes Service内に`spec.trafficDistribution`フィールドが導入されました。これにより、Serviceのエンドポイントにどのようにトラフィックをルーティングするかというpreferences を表現できるようになります。[トラフィックポリシー](https://kubernetes.io/docs/reference/networking/virtual-ips/#traffic-policies)が厳密なセマンティック保証にフォーカスしているのに対して、トラフィックディストリビューションでは(トポロジー的に近いエンドポイントへのルーティングなど) _preferences_ を表現することができます。これはパフォーマンス、コスト、信頼性の最適化に役立ちます。このフィールドを使用するには、クラスタとそのすべてのノードに対して `ServiceTrafficDistribution` フィーチャーゲートを有効にします。Kubernetes v1.30では、以下のフィールド値がサポートされています： 

`PreferClose`： クライアントにトポロジー的に近接したエンドポイントにトラフィックをルーティングする優先順位を示す。"topologically proximate" の解釈は実装によって異なり、同じノード、ラック、ゾーン、あるいはリージョン内のエンドポイントも含まれます。この値を設定することで、例えば負荷の均等な分散よりも近接性を最適化するなど、 実装が異なるトレードオフを行うことを許可できます。このようなトレードオフが許容できない場合、この値を設定すべきではありません。

このフィールドが設定されていない場合、実装（kube-proxyなど）はデフォルトのルーティング戦略を適用します。

詳細は[Traffic Distribution](https://kubernetes.io/docs/reference/networking/virtual-ips/#traffic-distribution)を参照してください。

### Storage Version Migration ([SIG API Machinery](https://github.com/kubernetes/community/tree/master/sig-api-machinery))

Kubernetes v1.30では、_StorageVersionMigration_ 用の新しい組み込みAPIが導入されました。Kubernetesは、保存ストレージに関連するいくつかのメンテナンス活動をサポートするために、APIデータが能動的に書き換えられることに依存しています。2つの顕著な例は、保存されたリソースのバージョン管理スキーマ（つまり、特定のリソースに対して優先されるストレージスキーマがv1からv2に変更されること）と、保存の暗号化（つまり、データの暗号化方法の変更に基づいて古いデータを書き換えること）です。

StorageVersionMigrationはアルファAPIで、以前は[out of tree](https://github.com/kubernetes-sigs/kube-storage-version-migrator)として提供されていました。

詳細は[storage version migration](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/storage-version-migration)を参照してください。


## Graduations, deprecations and removals for Kubernetes v1.30

### Graduated to stable

これは、安定版 (stable) (一般提供版 (general availability_) とも呼ばれます) に移行したすべての機能の一覧です。新機能やアルファ版からベータ版への移行を含むアップデートの全リストは、[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.30.md) を参照してください。

このリリースでは、合計 17 の機能が安定版 (Stable) に昇格しました。

- [Container Resource based Pod Autoscaling](https://kep.k8s.io/1610)
- [Remove transient node predicates from KCCM's service controller](https://kep.k8s.io/3458)
- [Go workspaces for k/k](https://kep.k8s.io/4402)
- [Reduction of Secret-based Service Account Tokens](https://kep.k8s.io/2799)
- [CEL for Admission Control](https://kep.k8s.io/3488)
- [CEL-based admission webhook match conditions](https://kep.k8s.io/3716)
- [Pod Scheduling Readiness](https://kep.k8s.io/3521)
- [Min domains in PodTopologySpread](https://kep.k8s.io/3022)
- [Prevent unauthorised volume mode conversion during volume restore](https://kep.k8s.io/3141)
- [API Server Tracing](https://kep.k8s.io/647)
- [Cloud Dual-Stack --node-ip Handling](https://kep.k8s.io/3705)
- [AppArmor support](https://kep.k8s.io/24)
- [Robust VolumeManager reconstruction after kubelet restart](https://kep.k8s.io/3756)
- [kubectl delete: Add interactive(-i) flag](https://kep.k8s.io/3895)
- [Metric cardinality enforcement](https://kep.k8s.io/2305)
- [Field `status.hostIPs` added for Pod](https://kep.k8s.io/2681)
- [Aggregated Discovery](https://kep.k8s.io/3352)

### Deprecations and removals

#### Removed the SecurityContextDeny admission plugin, deprecated since v1.27 ([SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth), [SIG Security](https://github.com/kubernetes/community/tree/master/sig-security), and [SIG Testing](https://github.com/kubernetes/community/tree/master/sig-testing))

SecurityContextDenyアドミッションプラグインが削除されたため、代わりにv1.25から利用できるPod Security Admissionプラグインが推奨されます。

## Release notes

Kubernetes v1.30リリースの全詳細は、我々の[リリースノート
ノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.30.md) をご覧ください。

## Availability

Kubernetes v1.30は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.30.0)からダウンロードできます。Kubernetesを使い始めるには、[対話型チュートリアル](https://kubernetes.io/docs/tutorials/)をチェックするか、[minikube](https://minikube.sigs.k8s.io/)を使ってローカルのKubernetesクラスタを実行します。[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)を使ってv1.30を簡単にインストールすることもできます。

## Release team

Kubernetesは、そのコミュニティのサポート、コミットメント、そしてハードワークがあって初めて可能になります。各リリースチームは、献身的なコミュニティのボランティアで構成されており、みなさんが信頼するKubernetesのリリースを構成する多くのパーツを協力して構築しています。これには、コードそのものからドキュメントやプロジェクト管理に至るまで、コミュニティのあらゆる場所にいる人々の専門的なスキルが必要です。

私たちのコミュニティにKubernetes v1.30リリースを提供するために懸命に働いてくれた[リリースチーム](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.30/release-team.md)全員に感謝したいと思います。リリースチームのメンバーは、初めてのシャドウから、数回のリリースサイクルで培った経験を持つ復帰したチームリーダーまで、多岐にわたります。リリースリーダーであるKat Cosgroveには、リリースサイクルの成功に向けて私たちをサポートし、私たちを擁護し、私たち全員が可能な限り最善の方法で貢献できるようにし、リリースプロセスの改善に向けて私たちに挑戦してくれたことに、特別な感謝を捧げます。

## Project velocity

CNCF K8s DevStatsプロジェクトは、Kubernetesと様々なサブプロジェクトのベロシティに関連する数多くの興味深いデータを集約しています。これは、個人のコントリビューションからコントリビューションしている企業の数まであらゆるものが含まれ、このエコシステムを進化させるために費やされている努力の深さと広さを示しています。

14週間（1月8日から4月17日）にわたって実施されたv1.30リリースサイクルでは、[863社](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.29.0%20-%20now&var-metric=contributions)と[1391人の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.29.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-repo_name=kubernetes%2Fkubernetes&var-country_name=All&var-companies=All)からの貢献がありました。


## Event update

* KubeCon + CloudNativeCon China 2024は、2024年8月21日～23日に香港で開催されます！カンファレンスの詳細と登録は[イベントサイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-open-source-summit-ai-dev-china/)をご覧ください。
* KubeCon + CloudNativeCon North America 2024は、2024年11月12日～15日に米国ユタ州ソルトレイクシティで開催されます！イベントサイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/)では、カンファレンスの詳細と参加登録をご覧いただけます。

## Upcoming release webinar

2024年5月23日（木）午前9時（PT）より、Kubernetes v1.30リリースチームのメンバーと一緒に、このリリースの主要機能、およびアップグレード計画に役立つ非推奨機能や削除機能について学びましょう。詳細と参加登録は、CNCF Online Programsサイトの[イベントページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-130-release/)をご覧ください。

## Get involved

Kubernetesに参加する最も簡単な方法は、あなたの興味に沿った多くの[Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs)の1つに参加することです。Kubernetesコミュニティに発信したいことがありますか？　私たちが毎週開催している[コミュニティミーティング](https://github.com/kubernetes/community/tree/master/communication)や以下のチャンネルであなたの声を共有してください。引き続きフィードバックとサポートをよろしくお願いします。

- 最新のアップデートは 𝕏 [@Kubernetesio](https://twitter.com/kubernetesio) でフォローしてください。
- [Discuss](https://discuss.kubernetes.io/)でコミュニティの議論に参加してください
- [Slack](http://slack.k8s.io/) でコミュニティに参加してください
- [Stack Overflow](http://stackoverflow.com/questions/tagged/kubernetes)に質問を投稿する
- あなたのKubernetesを共有する
  [ストーリー](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)
- [ブログ](https://kubernetes.io/blog/)でKubernetesの最新情報を読む
- [Kubernetesリリースチーム](https://github.com/kubernetes/sig-release/tree/master/release-team)から更に学ぶ

*このブログは2024年4月19日に更新され、当初リリースブログに含まれていなかった2つの追加変更にハイライトを当てています。*