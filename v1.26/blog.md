本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

原文:https://kubernetes.io/blog/2022/12/09/kubernetes-v1-26-release/

---

**Authors**: [Kubernetes 1.26 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.26/release-team.md)

It's with immense joy that we announce the release of Kubernetes v1.26!

Kubernetes v1.26 のリリースを発表できることを非常に嬉しく思います。

This release includes a total of 37 enhancements: eleven of them are graduating to Stable, ten are
graduating to Beta, and sixteen of them are entering Alpha. We also have twelve features being
deprecated or removed, three of which we better detail in this announcement.

このリリースは合計で37個のEnhancementがあります。11個がStableとなり、10個がBetaとなり、16個がAlphaとなりました。また、12個の機能がDeprecatedあるいは削除されました。この内3個をこのアナウンスの中で詳細に述べます。

## Release theme and logo 

**Kubernetes 1.26: Electrifying**

![Kubernetes 1.26 Electrifying logo](https://d33wubrfki0l68.cloudfront.net/852be6960c90223d8d573098bc3f0c16b56b2252/b1ba9/images/blog/2022-12-08-kubernetes-1.26-release/kubernetes-1.26.png)


The theme for Kubernetes v1.26 is _Electrifying_.

Kubernetes v1.26のテーマは _しびれること(Electrifying)_ です。

Each Kubernetes release is the result of the coordinated effort of dedicated volunteers, and only
made possible due to the use of a diverse and complex set of computing resources, spread out through
multiple datacenters and regions worldwide. The end result of a release - the binaries, the image
containers, the documentation - are then deployed on a growing number of personal, on-premises, and
cloud computing resources.

それぞれのKubernetesリリースは、献身的なボランティアによる強調した努力の結果です。
これは、世界中の複数のデータセンターと地域にある多様で複雑なコンピューティングリソースのセットを使用することで可能になりました。
リリースの最終成果物であるバイナリ、イメージコンテナ、ドキュメントは、ますます多くの個人利用、オンプレミス、およびクラウドコンピューティングリソースにデプロイされています。


In this release we want to recognise the importance of all these building blocks on which Kubernetes
is developed and used, while at the same time raising awareness on the importance of taking the
energy consumption footprint into account: 
environmental sustainability is an inescapable concern of creators and users of any software solution, and the environmental footprint of sofware, like Kubernetes, an area which we believe will play a significant role in future releases.

今回のリリースでは、Kubernetesの開発および利用の基盤となるこれらすべての構成要素の重要性を認識すると同時に、Kubernetesを利用する際には、次の点に留意する必要があります。
環境持続可能性(Environmental Sustainability)は、あらゆるソフトウェアソリューションの作成者と利用者にとって避けられない関心事です。
また、Kubernetesのようなソフトウェアの環境フットプリント(Environmental Footprint)は、今後のリリースで重要な役割を果たすと信じています。


As a community, we always work to make each new release process better than before (in this release,
we have [started to use Projects for tracking
enhancements](https://github.com/orgs/kubernetes/projects/98/views/1), for example). If [v1.24
"Stargazer"](/blog/2022/05/03/kubernetes-1-24-release-announcement/) _had us looking upwards, to
what is possible when our community comes together_, and [v1.25
"Combiner"](/blog/2022/08/23/kubernetes-v1-25-release/) _what the combined efforts of our community
are capable of_, this v1.26 "Electrifying" is also dedicated to all of those whose individual
motion, integrated into the release flow, made all of this possible.

コミュニティとして、私たちは常に新しいリリースのプロセスを以前より良くするために努力しています。（例えば、今回のリリースで私たちは[機能拡張の追跡にGitHubのProjectsの使用を開始しました](https://github.com/orgs/kubernetes/projects/98/views/1))。[v1.24 "Stargazer"](/blog/2022/05/03/kubernetes-1-24-release-announcement/) では _もし私たちが上を向いて、私たちのコミュニティが一緒になれば何ができるかを考えていました。_ そして[v1.25 "Combiner"](/blog/2022/08/23/kubernetes-v1-25-release/)では、 私たちのコミュニティが力を合わせると何ができるのか？_ を考えていました。
このv1.26 "Electrifying"もまた、リリースフローに統合された個人の動きで、私たちのコミュニティができることに貢献したすべての人に捧げます。

## Major themes
    
Kubernetes v1.26 is composed of many changes, brought to you by a worldwide team of volunteers. For
this release, we have identified several major themes.

Kubernetes v1.26は、世界中のボランティアチームによってもたらされた多くの変更で構成されています。今回のリリースでは、いくつかの主要なテーマがあります。

### Change in container image registry

In the previous release, [Kubernetes changed the container
registry](https://github.com/kubernetes/kubernetes/pull/109938), allowing the spread of the load
across multiple Cloud Providers and Regions, a change that reduced the reliance on a single entity
and provided a faster download experience for a large number of users.

前回のリリースで、[Kubernetesはコンテナレジストリ](https://github.com/kubernetes/kubernetes/pull/109938)を変更し、複数のクラウドプロバイダやリージョンに負荷を分散できるようにしました。1つのエンティティへの依存を削減し、多くのユーザーに対してより高速なダウンロード体験を提供するための変更でした。

This release of Kubernetes is the first that is exclusively published in the new `registry.k8s.io`
container image registry. In the (now legacy) `k8s.gcr.io` image registry, no container images tags
for v1.26 will be published, and only tags from releases before v1.26 will continue to be
updated. Refer to [registry.k8s.io: faster, cheaper and Generally
Available](/blog/2022/11/28/registry-k8s-io-faster-cheaper-ga/) for more information on the
motivation, advantages, and implications of this significant change.


今回のKubernetesのリリースは、はじめて`registry.k8s.io`に排他的に公開される最初のリリースです。(現在はレガシーな) `k8s.gcr.io` イメージレジストリでは、v1.26 用のコンテナイメージのタグは公開されません。v1.26 用のタグは発行されず、v1.26 より前のリリースのタグのみが引き続き更新されます。[registry.k8s.io:  faster, cheaper and GenerallyAvailable](/blog/2022/11/28/registry-k8s-io-faster-cheaper-ga/)を参照してください。
この重要な変更の動機、利点、および意味について、より詳しい情報を得ることができます。

### CRI v1alpha2 removed

With the adoption of the [Container Runtime Interface](/docs/concepts/architecture/cri/) (CRI) and
the [removal of dockershim](/blog/2022/02/17/dockershim-faq/) in v1.24, the CRI is the only
supported and documented way through which Kubernetes interacts with different container
runtimes. Each kubelet negotiates which version of CRI to use with the container runtime on that
node.

[Container Runtime Interface](/docs/concepts/architecture/cri/) (CRI)の採用や[dockershimの削除](/blog/2022/02/17/dockershim-faq/) がv1.24で実施されたことにより、CRIは、Kubernetesが異なるコンテナと対話するために唯一サポートされ、かつ文書化された方法となりました。各kubeletは、そのノード上のコンテナランタイムと使用するCRIのバージョンをネゴシエートします。

In the previous release, the Kubernetes project recommended using CRI version `v1`, but kubelet
could still negotiate the use of CRI `v1alpha2`, which was deprecated.

以前のリリースでは、KubernetesプロジェクトはCRIバージョン`v1`の使用を推奨していました。kubeletはまだ CRI `v1alpha2` の使用をネゴシエートできましたが、これは非推奨となりました。

Kubernetes v1.26 drops support for CRI `v1alpha2`.  That
[removal](https://github.com/kubernetes/kubernetes/pull/110618) will result in the kubelet not
registering the node if the container runtime doesn't support CRI `v1`. This means that containerd
minor version 1.5 and older are not supported in Kubernetes 1.26; if you use containerd, you will
need to upgrade to containerd version 1.6.0 or later **before** you upgrade that node to Kubernetes
v1.26. This applies equally to any other container runtimes that only support the `v1alpha2`: if
that affects you, you should contact the container runtime vendor for advice or check their website
for additional instructions in how to move forward.


Kubernetes v1.26 では、CRI `v1alpha2` のサポートが打ち切られました。 その
[削除](https://github.com/kubernetes/kubernetes/pull/110618) は、コンテナランタイムがCRI `v1` をサポートしていない場合、kubeletがノードを登録しない結果になります。
コンテナランタイムがCRI `v1` をサポートしていない場合、kubeletがノードを登録しないという結果になります。これは、containerdのマイナーバージョン1.5以前がKubernetes 1.26でサポートされないということです。containerd を使用する場合は、そのノードを Kubernetes にアップグレード **する前** に containerd をバージョン 1.6.0 またはそれ以降にアップグレードする必要があります。これは `v1alpha2` のみをサポートする他のコンテナランタイムにも同様に当てはまります。コンテナランタイムのベンダーに相談するか、そのウェブサイトをチェックしてください。

### Storage improvements

Following the GA of the [core Container Storage Interface (CSI)
Migration](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/625-csi-migration)
feature in the previous release, CSI migration is an on-going effort that we've been working on for
a few releases now, and this release continues to add (and remove) features aligned with the
migration's goals, as well as other improvements to Kubernetes storage.

CSIの移行は、以前のリリースに含まれていた[コアのコンテナ・ストレージ・インターフェイス（CSI）マイグレーション](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/625-csi-migration)のGAに続き、これまで数回のリリースで取り組んできた継続的な取り組みです。このリリースでは、Kubernetesストレージの他の改善と同様に、移行の目標に沿った機能を追加（および削除）しています。

#### CSI migration for Azure File and vSphere graduated to stable

Both the [vSphere](https://github.com/kubernetes/enhancements/issues/1491) and
[Azure](https://github.com/kubernetes/enhancements/issues/1885) in-tree driver migration to CSI have
graduated to Stable. You can find more information about them in the [vSphere CSI
driver](https://github.com/kubernetes-sigs/vsphere-csi-driver) and [Azure File CSI
driver](https://github.com/kubernetes-sigs/azurefile-csi-driver) repositories.

[vSphere](https://github.com/kubernetes/enhancements/issues/1491)と[Azure](https://github.com/kubernetes/enhancements/issues/1885)のインツリードライバのCSIへの移行は、Stableとなりました。これらに関する詳細な情報は、[vSphere CSI driver](https://github.com/kubernetes-sigs/vsphere-csi-driver) および [Azure File CSI driver](https://github.com/kubernetes-sigs/azurefile-csi-driver) リポジトリを参照してください。

#### _Delegate FSGroup to CSI Driver_ graduated to stable

This feature allows Kubernetes to [supply the pod's `fsGroup` to the CSI driver when a volume is
mounted](https://github.com/kubernetes/enhancements/issues/2317) so that the driver can utilize
mount options to control volume permissions.  Previously, the kubelet would always apply the
`fsGroup`ownership and permission change to files in the volume according to the policy specified in
the Pod's `.spec.securityContext.fsGroupChangePolicy` field.  Starting with this release, CSI
drivers have the option to apply the `fsGroup` settings during attach or mount time of the volumes.

この機能により、Kubernetesは[ボリュームのマウント時にPodの`fsGroup`をCSIドライバに供給し](https://github.com/kubernetes/enhancements/issues/2317)、ドライバがボリュームパーミッションを制御するためのマウントオプションを利用できるようにします。
以前は、kubeletは常にPodの`.spec.securityContext.fsGroupChangePolicy`フィールドに指定されたポリシーに従って、ボリューム内のファイルに `fsGroup` のオーナーシップとパーミッションの変更を適用していました。
このリリースから、CSIドライバーには、ボリュームのアタッチ時またはマウント時に `fsGroup` 設定を適用するオプションがあります。


#### In-tree GlusterFS driver removal

Already deprecated in the v1.25 release, the in-tree GlusterFS driver was
[removed](https://github.com/kubernetes/enhancements/issues/3446) in this release.

v1.25リリースですでに非推奨となっていた、インツリーのGlusterFSドライバは、このリリースで[削除](https://github.com/kubernetes/enhancements/issues/3446)されました。

#### In-tree OpenStack Cinder driver removal

This release removed the deprecated in-tree storage integration for OpenStack (the `cinder` volume
type). You should migrate to external cloud provider and CSI driver from
https://github.com/kubernetes/cloud-provider-openstack instead. For more information, visit [Cinder
in-tree to CSI driver migration](https://github.com/kubernetes/enhancements/issues/1489).

このリリースでは、OpenStack用の非推奨のインツリーのストレージ機能 (`cinder` ボリュームタイプ) が削除されました。外部クラウドプロバイダーとCSIドライバは、代わりに https://github.com/kubernetes/cloud-provider-openstack のサイトから移行してください。詳細については、[Cinder in-tree から CSI ドライバへの移行](https://github.com/kubernetes/enhancements/issues/1489) を参照してください。

### Signing Kubernetes release artifacts graduates to beta

Introduced in Kubernetes v1.24, [this
feature](https://github.com/kubernetes/enhancements/issues/3031) constitutes a significant milestone
in improving the security of the Kubernetes release process. All release artifacts are signed
keyless using [cosign](https://github.com/sigstore/cosign/), and both binary artifacts and images
[can be verified](https://kubernetes.io/docs/tasks/administer-cluster/verify-signed-artifacts/).

Kubernetes v1.24で導入された、[この機能](https://github.com/kubernetes/enhancements/issues/3031)は、Kubernetesのリリースプロセスのセキュリティを向上させる上で重要なマイルストーンとなります。すべてのリリースアーティファクトは[cosign](https://github.com/sigstore/cosign/)を使用してキーレスで署名され、バイナリアーティファクトとイメージの両方が[検証可能](https://kubernetes.io/docs/tasks/administer-cluster/verify-signed-artifacts/)です。


### Support for Windows privileged containers graduates to stable

Privileged container support allows containers to run with similar access to the host as processes
that run on the host directly. Support for this feature in Windows nodes, called [HostProcess
containers](/docs/tasks/configure-pod-container/create-hostprocess-pod/), will now [graduate to Stable](https://github.com/kubernetes/enhancements/issues/1981),
enabling access to host resources (including network resources) from privileged containers.

特権コンテナのサポートにより、コンテナはホスト上で直接実行されるプロセスと同様のアクセス権で実行することができます。Windowsノードでこの機能をサポートする、[HostProcess
コンテナ](/docs/tasks/configure-pod-container/create-hostprocess-pod/)が、[stableとなりました。](https://github.com/kubernetes/enhancements/issues/1981)特権コンテナからホストリソース(ネットワークリソースを含む)にアクセスできるようになります。

### Improvements to Kubernetes metrics

This release has several noteworthy improvements on metrics.

このリリースでは、メトリクスに関していくつかの注目すべき改善がなされています。

#### Metrics framework extension graduates to alpha

The metrics framework extension [graduates to
Alpha](https://github.com/kubernetes/enhancements/issues/3498), and
[documentation is now published for every metric in the
Kubernetes codebase](/docs/reference/instrumentation/metrics/).This enhancement adds two additional metadata
fields to Kubernetes metrics: `Internal` and `Beta`, representing different stages of metric maturity.

メトリックスフレームワークの拡張機能[がアルファとなりました](https://github.com/kubernetes/enhancements/issues/3498)。
[Kubernetesコードベースのすべてのメトリックについてドキュメントが公開されました](/docs/reference/instrumentation/metrics/)。
この拡張は、2つの追加のメタデータフィールドを追加します。それは`Internal`と`Beta`で、メトリックの成熟度の異なる段階を表しています。


#### Component Health Service Level Indicators graduates to alpha

Also improving on the ability to consume Kubernetes metrics, [component health Service Level
Indicators (SLIs)](/docs/reference/instrumentation/slis/) have [graduated to
Alpha](https://github.com/kubernetes/kubernetes/pull/112884): by enabling the `ComponentSLIs`
feature flag there will be an additional metrics endpoint which allows the calculation of Service
Level Objectives (SLOs) from raw healthcheck data converted into metric format.

また、Kubernetesのメトリクスを消費する機能が改善され、[component health Service Level Indicators (SLIs)](/docs/reference/instrumentation/slis/)が[アルファとなりました。](https://github.com/kubernetes/kubernetes/pull/112884) `ComponentSLIs` 機能フラグを有効にすることで、追加のメトリクスエンドポイントが利用できるようになります。それによりサービスレベルオブジェクティブ(SLO)の計算を可能にする追加のメトリクスエンドポイントが存在するようになります。


#### Feature metrics are now available

Feature metrics are now available for each Kubernetes component, making it possible to [track
whether each active feature gate is enabled](https://github.com/kubernetes/kubernetes/pull/112690)
by checking the component's metric endpoint for `kubernetes_feature_enabled`.

Kubernetesの各コンポーネントに対して機能メトリック（Feature metrics）が利用できるようになり、`kubernetes_feature_enabled` をチェックすることで、[各アクティブなフィーチャーゲートが有効かどうかを追跡する](https://github.com/kubernetes/kubernetes/pull/112690)ことが可能になりました。

### Dynamic Resource Allocation graduates to alpha

[Dynamic Resource
Allocation](/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)
is a [new feature](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/3063-dynamic-resource-allocation/README.md) 
that puts resource scheduling in the hands of third-party developers: it offers an
alternative to the limited "countable" interface for requesting access to resources
(e.g. `nvidia.com/gpu: 2`), providing an API more akin to that of persistent volumes. Under the
hood, it uses the [Container Device
Interface](https://github.com/container-orchestrated-devices/container-device-interface) (CDI) to do
its device injection. This feature is blocked by the `DynamicResourceAllocation` feature gate.


[ダイナミック・リソースアロケーション](/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)、サードパーティの開発者がリソースのスケジューリングを行えるようにする[新機能](/docs/cepts/scheduling-eviction/dynamic-resource-allocation/) です。
リソースへのアクセスを要求するための制限された "countable" インターフェースに代わるものです。
(例: `nvidia.com/gpu: 2`) リソースへのアクセスを要求するための限定的なより永続ボリュームに類似した API を提供します。内部実装としては、[Container Device Interface](CDI)(https://github.com/container-orchestrated-devices/container-device-interface)を使用して、デバイスインジェクションを行います。
この機能は `DynamicResourceAllocation` フィーチャーゲートでブロックされます。（アルファのため、デフォルトではOffということを言っていると思われる。）

### CEL in Admission Control graduates to alpha

[This feature](https://github.com/kubernetes/enhancements/issues/3488) introduces a `v1alpha1` API for [validating admission
policies](/docs/reference/access-authn-authz/validating-admission-policy/), enabling extensible admission
control via [Common Expression Language](https://github.com/google/cel-spec) expressions. Currently,
custom policies are enforced via [admission
webhooks](/docs/reference/access-authn-authz/extensible-admission-controllers/),
which, while flexible, have a few drawbacks when compared to in-process policy enforcement. To use,
enable the `ValidatingAdmissionPolicy` feature gate and the `admissionregistration.k8s.io/v1alpha1`
API via `--runtime-config`.

[この機能](https://github.com/kubernetes/enhancements/issues/3488) は [validating admission policies](/docs/reference/access-authn-authz/validating-admission-policy/) の`v1alpha1` API を導入します。
有効にすることで、[Common Expression Language](https://github.com/google/cel-spec)式による拡張可能なアドミッション制御を可能にします。現在ではカスタムポリシーは [admission webhooks](/docs/reference/access-authn-authz/extensible-admission-controllers/) を介して実施されています。
これは柔軟性がある一方で、プロセス内のポリシー適用と比較すると、いくつかの欠点があります。CELを使用するには`ValidatingAdmissionPolicy`フィーチャーゲートと APIの`admissionregistration.k8s.io/v1alpha1`を`--runtime-config` 経由で有効にします。

### Pod scheduling improvements

Kubernetes v1.26 introduces some relevant enhancements to the ability to better control scheduling
behavior.

Kubernetes v1.26では、スケジューリングの動作をより適切に制御できるよう、いくつかの関連する機能強化が導入されています。

#### `PodSchedulingReadiness` graduates to alpha

[This feature](https://github.com/kubernetes/enhancements/issues/3521) introduces a `.spec.schedulingGates` 
field to Pod's API, to [indicate whether the Pod is allowed to be scheduled or not](/docs/concepts/scheduling-eviction/pod-scheduling-readiness/). External users/controllers can use this field to hold a Pod from scheduling based on their policies and needs.

[この機能](https://github.com/kubernetes/enhancements/issues/3521)は、PodのAPIに `.spec.schedulingGates` フィールドを導入します。これによって[Podをスケジューリングから除外することを表現できます。](/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
外部ユーザー/コントローラーは、このフィールドを使用して、ポリシーとニーズに基づいてPodのスケジューリングを抑えることができます。


#### `NodeInclusionPolicyInPodTopologySpread` graduates to beta

By specifying a `nodeInclusionPolicy` in `topologySpreadConstraints`, you can control whether to
[take taints/tolerations into consideration](/docs/concepts/scheduling-eviction/topology-spread-constraints/)
when calculating Pod Topology Spread skew.

`topologySpreadConstraints` で `nodeInclusionPolicy` を指定することで、以下のことが制御可能になります。
Pod Topology Spreadのスキューを計算するときに[TaintやTolerationを考慮](/docs/concepts/scheduling-eviction/topology-spread-constraints/) するかどうかを制御できます。


## Other Updates

### Graduations to stable

This release includes a total of eleven enhancements promoted to Stable: 

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

12 features were [deprecated or removed](/blog/2022/11/18/upcoming-changes-in-kubernetes-1-26/) from
Kubernetes with this release.

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

The complete details of the Kubernetes v1.26 release are available in our [release
notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.26.md).

Kubernetes v1.26リリースの完全な詳細は、我々の[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.26.md)をご覧ください。

### Availability

Kubernetes v1.26 is available for download on [the Kubernetes site](https://k8s.io/releases/download/). 
To get started with Kubernetes, check out these [interactive tutorials](/docs/tutorials/) or run local 
Kubernetes clusters using containers as "nodes", with [kind](https://kind.sigs.k8s.io/). You can also 
easily install v1.26 using [kubeadm](/docs/setup/independent/create-cluster-kubeadm/).

Kubernetes v1.26は、[Kubernetesサイト](https://k8s.io/releases/download/)でダウンロードできます。
Kubernetesを使い始めるには、以下の[インタラクティブチュートリアル](/docs/tutorials/)をチェックアウトするか、またはローカルでコンテナを「ノード」として使用するKubernetesクラスタを[kind](https://kind.sigs.k8s.io/)で実行することもできます。
また [kubeadm](/docs/setup/independent/create-cluster-kubeadm/) を使って簡単にv1.26をインストールすることができます。

### Release team

Kubernetes is only possible with the support, commitment, and hard work of its community. Each release team is made up of dedicated community volunteers who work together to build the many pieces
that make up the Kubernetes releases you rely on. This requires the specialized skills of people from all corners of our community, from the code itself to its documentation and project management.

Kubernetesは、そのコミュニティのサポート、コミットメント、そしてハードワークによってのみ実現可能です。各リリースチームは、献身的なコミュニティのボランティアで構成され、皆さんが信頼するKubernetesのリリースを構成する多くの部品を一緒に構築しています。
これには、コードや文書化、プロジェクト管理まで、私たちのコミュニティの隅々にいる人々の専門的なスキルを必要とします。


We would like to thank the entire [release team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.26/release-team.md)
for the hours spent hard at work to ensure we deliver a solid Kubernetes v1.26 release for our community.

私たちは、コミュニティにKubernetes v1.26リリースを確実に提供するために多大な時間を費やしてくれた[リリースチーム](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.26/release-team.md)全体に感謝します。


A very special thanks is in order for our Release Lead, Leonard Pahlke, for successfully steering the entire release team throughout the entire release cycle, by making sure that we could all contribute in the best way possible to this release through his constant support and attention to the many and diverse details that make up the path to a successful release.

リリース・リードのLeonard Pahlkeには、リリース・サイクル全体を通して、私たちリリース・チーム全体をうまく舵取りしてくれたことに、特別な感謝を捧げたいと思います。
このリリースで私たち全員が最高の形で貢献できるよう、リリースを成功に導くための多くの様々なな事柄に常にサポートと配慮を怠りませんでした。

### User highlights
    
* Wortell faced increasingly higher ammounts of developer expertise and time for daily
  infrastructure management. [They used Dapr to reduce the complexity and amount of required
  infrastructure-related code, allowing them to focus more time on new
  features](https://www.cncf.io/case-studies/wortell/).

* Wortellは、開発者の専門知識と日々のインフラ管理に要する時間がますます増えていくことに直面していました。[Daprを使用することで、インフラ関連のコードの複雑さと必要な量を削減し、より多くの時間を新しいインフラ管理に集中できるようになりました。](https://www.cncf.io/case-studies/wortell/)

* Utmost handles sensitive personal data and needed SOC 2 Type II attestation, ISO 27001
  certification, and zero trust networking. [Using Cilium, they created automated pipelines that
  allowed developers to create new policies, supporting over 4,000 flows per
  second](https://www.cncf.io/case-studies/utmost/).

  * Utmostは、機密性の高い個人情報を取り扱うため、SOC 2 Type II認証、ISO 27001認証、ゼロ・トラストネットワーク認証が必要でした。[Ciliumを使用して、開発者が新しいポリシーを作成し、毎秒4,000以上のフローをサポートすることを可能にする自動パイプラインを作成しました。](https://www.cncf.io/case-studies/utmost/)


* Global cybersecurity company Ericom’s solutions depend on hyper-low latency and data
  security. [With Ridge's managed Kubernetes service they were able to deploy, through a single API,
  to a network of service providers worldwide](https://www.cncf.io/case-studies/ericom/).

* グローバルなサイバーセキュリティ企業であるEricom社のソリューションは、超低遅延とデータセキュリティに依存しています。[RidgeのマネージドKubernetesサービスにより、彼らは単一のAPIを通じて、世界中のサービスプロバイダのネットワークにデプロイすることができました。](https://www.cncf.io/case-studies/ericom/)


* Lunar, a Scandinavian online bank, wanted to implement quarterly production cluster failover
  testing to prepare for disaster recovery, and needed a better way to managed their platform
  services.[They started by centralizing their log management system, and followed-up with the
  centralization of all platform services, using Linkerd to connect the
  clusters](https://www.cncf.io/case-studies/lunar/).

* スカンジナビアのオンライン銀行である Lunarは、ディザスタリカバリに備え、四半期ごとに本番クラスタのフェイルオーバーテストを実施したいと考えていました。また、プラットフォームサービスを管理するためのより良い方法を必要としていました。
  [クラスターを接続するためにLinkerdを使用することで、ログ管理システムの一元化からスタートし、すべてのプラットフォームサービスを一元化しました。](https://www.cncf.io/case-studies/lunar/)


* Datadog runs 10s of clusters with 10,000+ of nodes and 100,000+ pods across multiple cloud
  providers.[They turned to Cilium as their CNI and kube-proxy replacement to take advantage of the
  power of eBPF and provide a consistent networking experience for their users across any
  cloud](https://www.cncf.io/case-studies/datadog/).

* Datadogは、複数のクラウドプロバイダーで10,000以上のノードと100,000以上のポッドを持つ数十のクラスタを実行しています。[CiliumをCNIとして使い kube-proxyの代替とし、eBPF のパワーを活用し、あらゆるクラウドでユーザーに一貫したネットワーク体験を提供することにしました。](https://www.cncf.io/case-studies/datadog/)

* Insiel wanted to update their software production methods and introduce a cloud native paradigm in
  their software production. [Their digital transformation project with Kiratech and Microsoft Azure
  allowed them to develop a cloud-first culture](https://www.cncf.io/case-studies/insiel/)
    

* Insielは、ソフトウェア制作の方法を更新し、クラウドネイティブパラダイムを導入することを望んでいました。[KiratechとMicrosoft Azureを使ったデジタルトランスフォーメーションプロジェクトにより、クラウドファーストの文化を発展させることができました。](https://www.cncf.io/case-studies/insiel/)

### Ecosystem updates

* KubeCon + CloudNativeCon Europe 2023 will take place in Amsterdam, The Netherlands, from 17 – 21
  April 2023! You can find more information about the conference and registration on the [event
  site](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/).

* KubeCon + CloudNativeCon Europe 2023 は、オランダのアムステルダムで 2023年4月17日 〜 21 日に開催されます。カンファレンスに関する詳細や参加登録は、[イベント
  サイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)をご覧ください。


* CloudNativeSecurityCon North America, a two-day event designed to foster collaboration, discussion
  and knowledge sharing of cloud native security projects and how to best use these to address
  security challenges and opportunities, will take place in Seattle, Washington (USA), from 1-2
  February 2023. See the [event
  page](https://events.linuxfoundation.org/cloudnativesecuritycon-north-america/) for more
  information.

* CloudNativeSecurityCon North Americaは、クラウドネイティブセキュリティプロジェクトのコラボレーション、ディスカッション、知識の共有し、セキュリティの課題と機会に対処するためにこれらをどのように活用するかを目的とした2日間のイベントです。アメリカのシアトルで2023年2月1日〜2日に開催されます。詳細は、[イベントページ](https://events.linuxfoundation.org/cloudnativesecuritycon-north-america/)をご覧ください。


* The CNCF announced [the 2022 Community Awards Winners](https://www.cncf.io/announcements/2022/10/28/cloud-native-computing-foundation-reveals-2022-community-awards-winners/): the Community Awards recognize CNCF community members that are going above and beyond to advance cloud native technology.

* CNCFは【2022年コミュニティアワード受賞者】(https://www.cncf.io/announcements/2022/10/28/cloud-native-computing-foundation-reveals-2022-community-awards-winners/)を発表しました。コミュニティアワードは、クラウドネイティブテクノロジーを発展させ、さらにそれ以上のことをしているCNCFコミュニティメンバーを表彰するものです。

### Project velocity

The [CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m) project
aggregates a number of interesting data points related to the velocity of Kubernetes and various
sub-projects. This includes everything from individual contributions to the number of companies that
are contributing, and is an illustration of the depth and breadth of effort that goes into evolving
this ecosystem.

[CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m)プロジェクトは、Kubernetesと様々なサブプロジェクトのベロシティに関連する多くの興味深いデータポイントを集約しています。これには、個人の貢献から、貢献している企業の数まで、あらゆるものが含まれています。このエコシステムを進化させるために費やされる努力の深さと広さを示しています。

In the v1.26 release cycle, which [ran for 14 weeks](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.26)
(September 5 to December 9), we saw contributions from [976 companies](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.25.0%20-%20v1.26.0&var-metric=contributions) and [6877 individuals](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.25.0%20-%20v1.26.0&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes).

v1.26 のリリースサイクルでは、[14 週間が費やされ](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.26)(9月5日から12月9日まで)、[976社](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.25.0%20-%20v1.26.0&var-metric=contributions) と [6877人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.25.0%20-%20v1.26.0&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes) からの貢献がありました。

## Upcoming Release Webinar

Join members of the Kubernetes v1.26 release team on Tuesday January 17, 2023 10am - 11am EST (3pm - 4pm UTC) to learn about the major features
of this release, as well as deprecations and removals to help plan for upgrades.  For more information and registration, visit the [event
page](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-v126-release/).

Kubernetes v1.26 リリースチームのメンバーが、2023年1月17日（火）10am - 11am EST (3pm - 4pm UTC) に、このリリースの主要機能についての情報を提供します。
このリリースの主な機能、およびアップグレードの計画に役立つ非推奨と削除について学びます。 詳細と参加登録は、[イベント
ページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-v126-release/)をご覧ください。


## Get Involved

The simplest way to get involved with Kubernetes is by joining one of the many [Special Interest
Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs) that align with your
interests.

Kubernetesに参加する最も簡単な方法は、あなたの興味に合った数多くの[Special Interest Group](https://github.com/kubernetes/community/blob/master/sig-list.md)(SIGs)のいずれかに参加することです。


Have something you’d like to broadcast to the Kubernetes community? Share your voice at our weekly
[community meeting](https://github.com/kubernetes/community/tree/master/communication), and through
the channels below:

Kubernetes コミュニティに発信したいことがありますか？毎週開催している[コミュニティ・ミーティング](https://github.com/kubernetes/community/tree/master/communication)、および以下のチャネルで、あなたの声を共有してください。

* Find out more about contributing to Kubernetes at the [Kubernetes
  Contributors](https://www.kubernetes.dev/) website

* Kubernetesへのコントリビューションについては、[Kubernetes Contributors](https://www.kubernetes.dev/)のウェブサイトをご覧ください。


* Follow us on Twitter [@Kubernetesio](https://twitter.com/kubernetesio) for the latest updates

* 最新情報はTwitter[@Kubernetesio](https://twitter.com/kubernetesio)でフォローしてください。


* Join the community discussion on [Discuss](https://discuss.kubernetes.io/)

* [Discuss](https://discuss.kubernetes.io/)のコミュニティディスカッションに参加してください。


* Join the community on [Slack](http://slack.k8s.io/)

* [Slack](http://slack.k8s.io/)のコミュニティに参加してください。


* Post questions (or answer questions) on [Server
  Fault](https://serverfault.com/questions/tagged/kubernetes)

* [Server Fault](https://serverfault.com/questions/tagged/kubernetes)に質問を投稿してください。(または質問に答えてください。)


*  [Share](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform) your Kubernetes story

* あなたのKubernetesストーリーを[共有](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)してください。


* Read more about what’s happening with Kubernetes on the [blog](https://kubernetes.io/blog/)

* Kubernetesで何が起こっているかについては、[ブログ](https://kubernetes.io/blog/)をお読みください。


* Learn more about the [Kubernetes Release
  Team](https://github.com/kubernetes/sig-release/tree/master/release-team)

* [Kubernetesリリース](https://github.com/kubernetes/sig-release/tree/master/release-team)の詳細については、こちらをご覧ください。
