### 本资源由 itjc8.com 收集整理
<p data-nodeid="16382">在前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=657&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6662&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="16386">《09 | 开关组件：如何使用功能开关，支持产品快速迭代》</a>那一讲中，我介绍过如何实现编译时开关和本地开关。有了这两种开关，我们就可以很方便地让测试人员在 App 里面手动启动或者关闭一些功能。那有没有什么好的办法可以让产品经理远程遥控功能呢？远程开关就能完成这一任务。</p>


<p data-nodeid="17038" class=""><strong data-nodeid="17043">通过远程开关，我们就可以在无须发布新版本的情况下开关 App 的某些功能，甚至可以为不同的用户群体提供不同的功能。</strong> 远程功能开关能帮助我们快速测试新功能，从而保证产品的快速迭代。</p>

<h3 data-nodeid="15404">远程功能开关模块的架构与实现</h3>
<p data-nodeid="15405">下面我们通过 Moments App 来看看如何架构一个灵活的远程功能开关模块，并使用 Firebase 来实现一个远程功能开关。该模块主要由两部分所组成：<strong data-nodeid="15497">Remote Config 模块</strong>和<strong data-nodeid="15498">Toggle 模块</strong>。远程功能开关模块的架构图如下所示：</p>
<p data-nodeid="17696" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/48/CioPOWCwv2SADMy0AAaS5zXqWdw226.png" alt="Drawing 0.png" data-nodeid="17699"></p>

<h4 data-nodeid="18356" class="">1. Remote Config 模块的架构与实现</h4>

<p data-nodeid="15410">由于 Toggle 模块依赖于 Remote Config 模块，所以我们就先看一下 Remote Config 模块的架构与实现。</p>
<p data-nodeid="19010" class=""><strong data-nodeid="19015">Remote Config 也叫作“远程配置”，它可以帮助我们把 App 所需的配置信息存储在服务端，让所有的 App 在启动的时候读取相关的配置信息，并根据这些配置信息来调整 App 的行为。</strong> Remote Config 应用广泛，可用于远程功能开关、 A/B 测试和强制更新等功能上。</p>

<p data-nodeid="15412">Remote Config 的架构十分简单，由<code data-backticks="1" data-nodeid="15507">RemoteConfigKey</code>和<code data-backticks="1" data-nodeid="15509">RemoteConfigProvider</code>所组成，其中<code data-backticks="1" data-nodeid="15511">RemoteConfigKey</code>是一个空协议（Protocol），用于存放配置信息的唯一标识，其定义如下：</p>
<pre class="lang-swift" data-nodeid="15413"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">protocol</span> <span class="hljs-title">RemoteConfigKey</span> </span>{ }
</code></pre>
<p data-nodeid="15414">为了支持 Firebase 的 Remote Config 服务，我们定义一个遵循了<code data-backticks="1" data-nodeid="15514">RemoteConfigKey</code>协议的枚举类型（Enum）， 其具体的代码如下：</p>
<pre class="lang-swift" data-nodeid="15415"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">FirebaseRemoteConfigKey</span>: <span class="hljs-title">String</span>, <span class="hljs-title">RemoteConfigKey</span> </span>{
    <span class="hljs-keyword">case</span> isRoundedAvatar
}
</code></pre>
<p data-nodeid="15416">因为 Firebase Remote Config 的标识都是字符串类型，所以我们把<code data-backticks="1" data-nodeid="15517">FirebaseRemoteConfigKey</code>的<code data-backticks="1" data-nodeid="15519">rawValue</code>也指定为<code data-backticks="1" data-nodeid="15521">String</code>类型，这样就能很方便地取出<code data-backticks="1" data-nodeid="15523">case</code>的值，例如，通过<code data-backticks="1" data-nodeid="15525">FirebaseRemoteConfigKey.isRoundedAvatar.rawValue</code>来得到“isRoundedAvatar”字符串。</p>
<p data-nodeid="15417">有了配置信息的标识以后，我们再来看看如何在 App 里面访问 Remote Config 服务。首先，我们定义一个名叫<code data-backticks="1" data-nodeid="15528">RemoteConfigProvider</code>的协议，其定义如下：</p>
<pre class="lang-swift" data-nodeid="15418"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">protocol</span> <span class="hljs-title">RemoteConfigProvider</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">setup</span><span class="hljs-params">()</span></span>
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">fetch</span><span class="hljs-params">()</span></span>
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getString</span><span class="hljs-params">(by key: RemoteConfigKey)</span></span> -&gt; <span class="hljs-type">String?</span>
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getInt</span><span class="hljs-params">(by key: RemoteConfigKey)</span></span> -&gt; <span class="hljs-type">Int?</span>
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getBool</span><span class="hljs-params">(by key: RemoteConfigKey)</span></span> -&gt; <span class="hljs-type">Bool</span>
}
</code></pre>
<p data-nodeid="15419"><code data-backticks="1" data-nodeid="15530">RemoteConfigProvider</code>协议定义了<code data-backticks="1" data-nodeid="15532">setup()</code>、<code data-backticks="1" data-nodeid="15534">fetch()</code>等五个方法。为了使用 Firebase 的Remote Config 服务，我们定义了一个结构体<code data-backticks="1" data-nodeid="15536">FirebaseRemoteConfigProvider</code>来遵循该协议，该结构体实现了协议里的五个方法。</p>
<p data-nodeid="15420">我们先来看一下<code data-backticks="1" data-nodeid="15539">setup()</code>和<code data-backticks="1" data-nodeid="15541">fetch()</code>方法的具体代码实现：</p>
<pre class="lang-swift" data-nodeid="15421"><code data-language="swift"><span class="hljs-keyword">private</span> <span class="hljs-keyword">let</span> remoteConfig = <span class="hljs-type">RemoteConfig</span>.remoteConfig()
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">setup</span><span class="hljs-params">()</span></span> {
    remoteConfig.setDefaults(fromPlist: <span class="hljs-string">"FirebaseRemoteConfigDefaults"</span>)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">fetch</span><span class="hljs-params">()</span></span> {
    remoteConfig.fetchAndActivate()
}
</code></pre>
<p data-nodeid="15422">在初始化的时候，我们调用 Firebase SDK 所提供的<code data-backticks="1" data-nodeid="15544">RemoteConfig.remoteConfig()</code>方法来生成一个<code data-backticks="1" data-nodeid="15546">RemoteConfig</code>的实例并赋值给<code data-backticks="1" data-nodeid="15548">remoteConfig</code>属性，然后在<code data-backticks="1" data-nodeid="15550">setup()</code>里调用<code data-backticks="1" data-nodeid="15552">remoteConfig.setDefaults(fromPlist:)</code>方法从 FirebaseRemoteConfigDefaults.plist 文件里读取配置的默认值。下图展示的就是该 plist 文件，在该文件里，我们把<code data-backticks="1" data-nodeid="15554">isRoundedAvatar</code>的默认值设置为 false，这样能保证 App 在无法联网的情况下也能正常运行。</p>
<p data-nodeid="19670" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/48/CioPOWCwv3SAVlZxAAEErVPTcjU511.png" alt="Drawing 1.png" data-nodeid="19673"></p>

<p data-nodeid="15424">在<code data-backticks="1" data-nodeid="15558">fetch()</code>里，我们调用了 Firebase SDK 里的<code data-backticks="1" data-nodeid="15560">fetchAndActivate()</code>方法来获取远程配置信息。</p>
<p data-nodeid="15425">接着我们再来看看另外三个方法的具体实现：</p>
<pre class="lang-swift" data-nodeid="15426"><code data-language="swift"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getString</span><span class="hljs-params">(by key: RemoteConfigKey)</span></span> -&gt; <span class="hljs-type">String?</span> {
    <span class="hljs-keyword">guard</span> <span class="hljs-keyword">let</span> key = key <span class="hljs-keyword">as</span>? <span class="hljs-type">FirebaseRemoteConfigKey</span> <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
    }
    <span class="hljs-keyword">return</span> remoteConfig[key.rawValue].stringValue
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getInt</span><span class="hljs-params">(by key: RemoteConfigKey)</span></span> -&gt; <span class="hljs-type">Int?</span> {
    <span class="hljs-keyword">guard</span> <span class="hljs-keyword">let</span> key = key <span class="hljs-keyword">as</span>? <span class="hljs-type">FirebaseRemoteConfigKey</span> <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
    }
    <span class="hljs-keyword">return</span> <span class="hljs-type">Int</span>(truncating: remoteConfig[key.rawValue].numberValue)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getBool</span><span class="hljs-params">(by key: RemoteConfigKey)</span></span> -&gt; <span class="hljs-type">Bool</span> {
    <span class="hljs-keyword">guard</span> <span class="hljs-keyword">let</span> key = key <span class="hljs-keyword">as</span>? <span class="hljs-type">FirebaseRemoteConfigKey</span> <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>
    }
    <span class="hljs-keyword">return</span> remoteConfig[key.rawValue].boolValue
}
</code></pre>
<p data-nodeid="15427">这三个方法都使用了<code data-backticks="1" data-nodeid="15564">RemoteConfigKey</code>作为标识符从<code data-backticks="1" data-nodeid="15566">remoteConfig</code>对象里读取相关的配置信息，然后把获取到的信息分别转换成所需的类型，例如字符串、整型或者布尔类型。</p>
<p data-nodeid="15428">至此，我们就实现了 Remote Config 模块，假如还需要支持其他的远程配置服务，只需为<code data-backticks="1" data-nodeid="15569">RemoteConfigProvider</code>协议实现另外一个子类型即可，例如需要支持 Optimizely 的远程配置服务时，可以实现一个名叫<code data-backticks="1" data-nodeid="15571">OptimizelyRemoteConfigProvider</code>的结构体来封装访问 Optimizely 后台服务的逻辑。</p>
<h4 data-nodeid="20332" class="">2. Toggle 模块的架构与实现</h4>

<p data-nodeid="15432">有了 Remote Config 模块，实现 Toggle 模块就变得十分简单了。在前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=657&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6662&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="15577">《09 | 开关组件：如何使用功能开关，支持产品快速迭代》</a>里面，我们讲过 Toggle 模块的架构与实现。要添加远程开关的支持，我们只需要增加两个实现类型：<code data-backticks="1" data-nodeid="15579">RemoteToggle</code>和<code data-backticks="1" data-nodeid="15581">FirebaseRemoteTogglesDataStore</code>结构体。我们先看一下<code data-backticks="1" data-nodeid="15583">RemoteToggle</code>的实现：</p>
<pre class="lang-swift" data-nodeid="15433"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">RemoteToggle</span>: <span class="hljs-title">String</span>, <span class="hljs-title">ToggleType</span> </span>{
    <span class="hljs-keyword">case</span> isRoundedAvatar
}
</code></pre>
<p data-nodeid="15434">和编译时开关以及本地开关一样，<code data-backticks="1" data-nodeid="15586">RemoteToggle</code>也是一个遵循了<code data-backticks="1" data-nodeid="15588">ToggleType</code>协议的枚举类型。所有的远程开关功能的名称都罗列在<code data-backticks="1" data-nodeid="15590">case</code>里面，例如，<code data-backticks="1" data-nodeid="15592">isRoundedAvatar</code>表示是否把朋友圈页面里的头像显示为圆形。</p>
<p data-nodeid="15435">有了功能开关的名称定义以后，我们就要为<code data-backticks="1" data-nodeid="15595">TogglesDataStoreType</code>提供一个远程开关的具体实现。因为我们使用了 Firebase 服务，所以就把它命名为<code data-backticks="1" data-nodeid="15597">FirebaseRemoteTogglesDataStore</code>，其具体实现如下：</p>
<pre class="lang-swift" data-nodeid="15436"><code data-language="swift"><span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">FirebaseRemoteTogglesDataStore</span>: <span class="hljs-title">TogglesDataStoreType</span> </span>{
    <span class="hljs-keyword">static</span> <span class="hljs-keyword">let</span> shared: <span class="hljs-type">FirebaseRemoteTogglesDataStore</span> = .<span class="hljs-keyword">init</span>()
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">let</span> remoteConfigProvider: <span class="hljs-type">RemoteConfigProvider</span>
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">init</span>(remoteConfigProvider: <span class="hljs-type">RemoteConfigProvider</span> = <span class="hljs-type">FirebaseRemoteConfigProvider</span>.shared) {
        <span class="hljs-keyword">self</span>.remoteConfigProvider = remoteConfigProvider
        <span class="hljs-keyword">self</span>.remoteConfigProvider.setup()
        <span class="hljs-keyword">self</span>.remoteConfigProvider.fetch()
    }
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">isToggleOn</span><span class="hljs-params">(<span class="hljs-number">_</span> toggle: ToggleType)</span></span> -&gt; <span class="hljs-type">Bool</span> {
        <span class="hljs-keyword">guard</span> <span class="hljs-keyword">let</span> toggle = toggle <span class="hljs-keyword">as</span>? <span class="hljs-type">RemoteToggle</span>, <span class="hljs-keyword">let</span> remoteConfiKey = <span class="hljs-type">FirebaseRemoteConfigKey</span>(rawValue: toggle.rawValue) <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>
        }
        <span class="hljs-keyword">return</span> remoteConfigProvider.getBool(by: remoteConfiKey)
    }
    <span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">update</span><span class="hljs-params">(toggle: ToggleType, value: Bool)</span></span> { }
}
</code></pre>
<p data-nodeid="15437"><code data-backticks="1" data-nodeid="15599">FirebaseRemoteTogglesDataStore</code>依赖了 Remote Config 模块。在<code data-backticks="1" data-nodeid="15601">init()</code>方法里面，我们通过依赖注入的方式把<code data-backticks="1" data-nodeid="15603">FirebaseRemoteConfigProvider</code>的实例传递进来，并调用<code data-backticks="1" data-nodeid="15605">setup()</code>方法来初始化 Firebase 的 Remote Config 服务，然后调用<code data-backticks="1" data-nodeid="15607">fetch()</code>方法来读取所有的配置信息。</p>
<p data-nodeid="15438">因为<code data-backticks="1" data-nodeid="15610">FirebaseRemoteTogglesDataStore</code>遵循了<code data-backticks="1" data-nodeid="15612">TogglesDataStoreType</code>协议，所以必须实现<code data-backticks="1" data-nodeid="15614">isToggleOn(_:)</code>和<code data-backticks="1" data-nodeid="15616">update(toggle:value:)</code>两个方法。</p>
<p data-nodeid="15439"><code data-backticks="1" data-nodeid="15618">isToggleOn(_:)</code>方法用于判断某个开关是否打开，在方法实现里，我们先判断传递进来的<code data-backticks="1" data-nodeid="15620">toggle</code>是否为<code data-backticks="1" data-nodeid="15622">RemoteToggle</code>类型，然后再判断该 Toggle 的名称是否匹配<code data-backticks="1" data-nodeid="15624">FirebaseRemoteConfigKey</code>里的定义。如果都符合条件，那么就可以调用<code data-backticks="1" data-nodeid="15626">remoteConfigProvider</code>的<code data-backticks="1" data-nodeid="15628">getBool(by:)</code>方法来判断开关是否打开。</p>
<p data-nodeid="15440"><code data-backticks="1" data-nodeid="15630">update(toggle: ToggleType, value: Bool)</code>方法的实现非常简单，因为 App 是无法更新远程开关信息的，所以它的实现为空。</p>
<p data-nodeid="15441">至此，我们就为 Toggle 模块添加好了 Firebase 远程开关的支持。</p>
<h3 data-nodeid="15442">远程开关的使用与配置</h3>
<p data-nodeid="15443"><strong data-nodeid="15640">使用远程开关仅仅需要两步</strong>，下面我们就以<code data-backticks="1" data-nodeid="15638">MomentListItemView</code>为例子看看如何使用远程开关来控制头像的显示风格吧。</p>
<p data-nodeid="15444">第一步是在<code data-backticks="1" data-nodeid="15642">init()</code>方法里面把<code data-backticks="1" data-nodeid="15644">TogglesDataStoreType</code>子类型的实例通过依赖注入的方式传递进去，具体代码如下：</p>
<pre class="lang-java" data-nodeid="15445"><code data-language="java"><span class="hljs-keyword">private</span> let remoteTogglesDataStore: <span class="hljs-function">TogglesDataStoreType
<span class="hljs-title">init</span><span class="hljs-params">(frame: CGRect = .zero, ..., remoteTogglesDataStore: TogglesDataStoreType = FirebaseRemoteTogglesDataStore.shared)</span> </span>{
    self.remoteTogglesDataStore = remoteTogglesDataStore
    <span class="hljs-keyword">super</span>.init(frame: frame)
}
</code></pre>
<p data-nodeid="15446">因为 Moments App 使用了 Firebase 作为远程开关服务，所以我们就把<code data-backticks="1" data-nodeid="15647">FirebaseRemoteTogglesDataStore</code>的实例赋值给<code data-backticks="1" data-nodeid="15649">remoteTogglesDataStore</code>属性。</p>
<p data-nodeid="15447">第二步是调用<code data-backticks="1" data-nodeid="15652">isToggleOn(_:)</code>方法来判断远程开关是否开启，示例代码如下：</p>
<pre class="lang-swift" data-nodeid="15448"><code data-language="swift"><span class="hljs-keyword">if</span> remoteTogglesDataStore.isToggleOn(<span class="hljs-type">RemoteToggle</span>.isRoundedAvatar) {
    avatarImageView.asAvatar(cornerRadius: <span class="hljs-number">10</span>)
}
</code></pre>
<p data-nodeid="15449">我们把<code data-backticks="1" data-nodeid="15655">isRoundedAvatar</code>作为标识符来调用<code data-backticks="1" data-nodeid="15657">isToggleOn(_:)</code>方法，如果该方法返回<code data-backticks="1" data-nodeid="15659">true</code>，就把<code data-backticks="1" data-nodeid="15661">avatarImageView</code>的圆角设置为 10 pt。因为<code data-backticks="1" data-nodeid="15663">avatarImageView</code>的高度和宽度都为 20 pt，所以当圆角设置为 10 pt 时就会显示为圆形。</p>
<p data-nodeid="15450">就这样，我们就能在 App 里使用名为<code data-backticks="1" data-nodeid="15666">isRoundedAvatar</code>的远程开关了。假如要使用其他的远程开关，只需要在<code data-backticks="1" data-nodeid="15668">RemoteToggle</code>和<code data-backticks="1" data-nodeid="15670">FirebaseRemoteConfigKey</code>两个枚举类型里添加新的<code data-backticks="1" data-nodeid="15672">case</code>，并在 FirebaseRemoteConfigDefaults.plist 文件设置默认值即可。</p>
<p data-nodeid="15451">但是，产品经理怎样才能在 Firebase 服务端<strong data-nodeid="15679">配置远程开关</strong>呢？下面我们一起看一下这个配置的步骤吧。</p>
<p data-nodeid="20988" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/40/Cgp9HWCwv4mAaXxBAAFQQUUchME827.png" alt="Drawing 2.png" data-nodeid="20991"></p>

<p data-nodeid="15453">我们可以在 Firebase 网站上点击 Engage -&gt;  Remote Config 菜单来打开 Remote Config 配置页面，然后点击“Add parameter”来添加一个名叫“isRoundedAvatar”的配置，如下图所示：</p>
<p data-nodeid="21650" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/40/Cgp9HWCwv46AJnkoAABDkckTx4Q957.png" alt="Drawing 3.png" data-nodeid="21653"></p>

<p data-nodeid="15455">当添加或修改完配置后，一定要记住点击下图的“Publish changes”按钮来发布更新。</p>
<p data-nodeid="22316" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/48/CioPOWCwv5WAeslvAAA35s5FG6I535.png" alt="Drawing 4.png" data-nodeid="22319"></p>

<p data-nodeid="15457">现在我们就能很方便地在 Firebase 网站上修改“isRoundedAvatar”配置的值来控制头像的显示风格了。</p>
<p data-nodeid="15458">除了简单地启动或者关闭远程开关以外，Firebase 还可以帮我们根据用户的特征进行条件配置，例如，我们可以让所有使用中文的用户启动圆形头像风格，而让其他语言的用户保留原有风格。</p>
<p data-nodeid="15459">下面我们就来看看如何在 Firebase 网站上<strong data-nodeid="15692">进行条件配置</strong>。</p>
<p data-nodeid="15460">我们可以点击修改按钮的图标来打开修改弹框，然后点击“Add value for condition”按钮来添加条件。如下图所示，我们添加了一个名叫“Chinese users”的条件，该条件会判断用户是否使用中文作为他们设备的默认语言。</p>
<p data-nodeid="22986" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/48/CioPOWCwv5uAVDBQAABwa1bn3zE273.png" alt="Drawing 5.png" data-nodeid="22989"></p>

<p data-nodeid="15462">然后我们就可以为符合该条件的用户配置不同的值，例如在下图中，符合“Chinese users”条件的用户在读取“isRoundedAvatar”配置时都会得到<code data-backticks="1" data-nodeid="15696">true</code>。</p>
<p data-nodeid="23660" class=""><img src="https://s0.lgstatic.com/i/image6/M00/42/48/CioPOWCwv6CAVm2nAABefStzJKo444.png" alt="Drawing 6.png" data-nodeid="23663"></p>

<p data-nodeid="15464">下面是 Moments App 运行在不同语言设备上的效果图，你可以对比一下。</p>
<p data-nodeid="25696"><img src="https://s0.lgstatic.com/i/image6/M01/42/40/Cgp9HWCwv6mALnzGAGJIzKWo3xc607.png" alt="Drawing 7.png" data-nodeid="25699"></p>



<h3 data-nodeid="15467">总结</h3>
<p data-nodeid="15468">在这一讲中，我们主要讲解了如何架构一个灵活的远程开关模块，该模块可以使用不同的后台服务来支持远程开关。接着我们以 Firebase 作为例子讲述了如何使用 Remote Config 来实现一个头像风格的远程开关，并且演示了如何根据用户的特征来为远程开关配置不同的值。</p>
<p data-nodeid="15469">有了远程开关，产品经理就能很方便地遥控 App 的行为，并能快速地尝试新功能。但需要注意的是：<strong data-nodeid="15708">不能滥用远程开关，并且最好能经常回顾上线的远程开关，把测试完毕的开关及时删除掉</strong>，否则会导致 App 里面的开关越来越多，使得程序的逻辑变得十分复杂且难以维护，再加上每个远程开关都需要从网络读取相关的配置信息，太多的开关还会影响到用户的使用体验。</p>
<p data-nodeid="15470"><strong data-nodeid="15712">思考题</strong></p>
<blockquote data-nodeid="15471">
<p data-nodeid="15472">在 FirebaseRemoteTogglesDataStore 里面，为什么没有直接使用 Firebase SDK 来读取 Remote Config 呢？另外，把读取 Remote Config 的逻辑封装在 FirebaseRemoteConfigProvider 里有什么好处呢？</p>
</blockquote>
<p data-nodeid="15473">可以把你的思考与答案写到留言区哦。下一讲我将介绍“如何使用 A/B 测试协助产品抉择”的相关内容，记得按时来听课哦。</p>
<p data-nodeid="15474"><strong data-nodeid="15718">源码地址</strong></p>
<blockquote data-nodeid="15475">
<p data-nodeid="15476">RemoteConfig 源码地址：<a href="https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Foundations/RemoteConfig?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="15722">https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Foundations/RemoteConfig</a><br>
远程开关源码地址：<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Moments/Moments/Foundations/Toggles/FirebaseRemoteTogglesDataStore.swift?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="15727">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/Moments/Moments/Foundations/Toggles/FirebaseRemoteTogglesDataStore.swift</a></p>
</blockquote>

---

### 精选评论

##### **超：
> 1.这种开关本地plist存的往往是false, 如果远程请求失败，是不是意味着永远只能用本地的false了？2.如果是一个很重要的开关设置在app 启动的时候，这个时候难道要等待远程数据返回吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1.你的理解是对的，但不可能一直都连接不上网络哦哦。2.这个开关会在第二次才生效哦，所以很重要的功能不应该放远程开关，而是事先测试好才发布。

##### **用户3586：
> 老师好：关于配置准备：如何搭建多环境支持，为 App 开发作准备 的问题我的需求是：我的需求就是一个target多个scheme对应多个config，根据不通过的config输出不同的app（功能大小差异而已）我用xcconfig来配置工程，在a.xcconfig中PROVISIONING_PROFILE_SPECIFIER=a.mobileprovisionPRODUCT_BUNDLE_ID = com.test.a在b.xcconfig中PROVISIONING_PROFILE_SPECIFIER=b.mobileprovisionPRODUCT_BUNDLE_ID = com.test.b我当前切换到b版本archive后mobileprovision还是用的a.mobileprovision，这是需要咋设置？我现在版本是a版本它的PRODUCT_BUNDLE_ID = com.test.a， 切换到b版本后PRODUCT_BUNDLE_ID = com.test.b，archive后bundid还是PRODUCT_BUNDLE_ID = com.test.a 没有变? 我必须手动来改这个bundid，打包才会正确，我下载的你demo发现也有这个问题，你可以自己试试看？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦哦，我的 App 是一个 Target 对应一个 Scheme。一个 target 一个 bundle id。这样就可以保证打包全自动了，可以看看后面的 《第 25 讲| 自动化构建：解决大量重复性人力工作神器 》和 《第 26 讲| 持续集成：如何实现无需人手的快速交付？》

##### **星：
> 在网站上更新配置文件，代码加载本地plist文件是怎么读到新配置的呢，不太明白。请老师解答

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，plist 文件里面是默认值，从网站更新的值是 Firebase 服务启动的时候自动读取的并自动缓存起来，它不会更新 plist，等于 App 重新安装，或者清除缓存以后又重新读 plist 的默认值了。

