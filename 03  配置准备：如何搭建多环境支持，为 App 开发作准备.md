### 本资源由 itjc8.com 收集整理
<p data-nodeid="10683" class="">在开始之前，我先问你几个问题，在测试的时候，App 一般需要连接测试服务器，那么在上架后，还需要连生产服务器吗？在发布前，你的 App 需要通过 Ad-hoc 分发给内部测试组吗？在发布到 App Store 的时候，你的 App 需要同时支持免费版和收费版吗？</p>
<p data-nodeid="10684">如果你的回答是“是”，那么你的 App 就需要搭建多环境支持，优化开发的工作流程。多环境提供很多好处，比如能基于同一套源代码自动构建出有差异功能的 App；能支持多个团队并行开发，也能分离测试和生产环境，提高产品的迭代速度，保证上架的 App 通过严格测试和功能验证。</p>
<p data-nodeid="10685">在 Moments App 项目中，我们就使用了三个不同的环境，分别是开发环境，测试环境和生产环境。它们到底有什么区别呢？</p>
<ul data-nodeid="10686">
<li data-nodeid="10687">
<p data-nodeid="10688"><strong data-nodeid="10858">开发环境，</strong> 用于日常的开发，一般有未完成的功能模块。编译时，也不进行任何优化，可以打印更多的日志，帮助开发者快速定位问题。</p>
</li>
<li data-nodeid="10689">
<p data-nodeid="10690"><strong data-nodeid="10863">测试环境，</strong> 主要是用于测试，以及为产品经理进行功能验证，包括部分完成的功能模块，也提供一些隐藏功能，方便我们进行开发和迭代，例如快速切换用户，清理 Cache，连接到不同后台服务器等等。</p>
</li>
<li data-nodeid="10691">
<p data-nodeid="10692"><strong data-nodeid="10868">生产环境，</strong> 只包含通过了测试并验证过的功能模块，它是最终提交到 App Store 供终端用户使用的版本。</p>
</li>
</ul>
<p data-nodeid="10693">多环境支持需要用到 Xcode 的构建配置，这一讲，我就结合 Moments App 项目来聊聊这个问题。</p>
<h3 data-nodeid="10694">Xcode 构建基础概念</h3>
<p data-nodeid="10695">一般在构建一个 iOS App 的时候，需要用到 Xcode Project，Xcode Target，Build Settings，Build Configuration 和 Xcode Scheme 等构建配置。它们各有什么用呢？</p>
<h4 data-nodeid="10696">Xcode Project</h4>
<p data-nodeid="10697"><strong data-nodeid="10877">Xcode Project</strong>用于组织源代码文件和资源文件。一个 Project 可以包含多个 Target，例如当我们新建一个 Xcode Project 的时候，它会自动生成 App 的主 Target，Unit Test Target 和 UI Test Target。</p>
<p data-nodeid="10698">在 Moments App 项目中，主 Target 就是 Moments，Unit Test Target 是 MomentsTests， UI Test Target 就是 MomentsUITests。</p>
<p data-nodeid="10699"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9Eb2ARf3OAAK1tf7-bjM651.png" alt="Drawing 1.png" data-nodeid="10881"></p>
<h4 data-nodeid="10700">Xcode Target</h4>
<p data-nodeid="10701"><strong data-nodeid="10887">Xcode Target</strong>用来定义如何构建出一个产品（例如 App， Extension 或者 Framework），Target 可以指定需要编译的源代码文件和需要打包的资源文件，以及构建过程中的步骤。</p>
<p data-nodeid="10702">例如在我们的 Moments App 项目中，负责单元测试的<strong data-nodeid="10897">MomentsTests</strong>Target 就指定了 14 个测试文件需要构建（见下图的 Compile Sources），并且该 Target 依赖了主 App Target<strong data-nodeid="10898">Moments</strong>（见下图的 Dependencies）。</p>
<p data-nodeid="10703"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9EciAH1boAAaiePlJeSA718.png" alt="Drawing 3.png" data-nodeid="10901"></p>
<p data-nodeid="10704">有了 Target 的定义，构建系统就可以读取相关的源代码文件进行编译，然后把相关的资源文件进行打包，并严格按照 Target 所指定的设置和步骤执行。那么 Target 所指定的设置哪里来的呢？来自 Build Settings。</p>
<h4 data-nodeid="10705">Build Settings</h4>
<p data-nodeid="10706"><strong data-nodeid="10908">Build Setting</strong>保存了构建过程中需要用到的信息，它以一个变量的形式而存在，例如所支持的设备平台，或者支持操作系统的最低版本等。</p>
<p data-nodeid="10707">通常，一条 Build Setting 信息由两部分组成：名字和值。比如下面是一条 Setting 信息，<code data-backticks="1" data-nodeid="10910">iOS Development Target</code>是名字，而<code data-backticks="1" data-nodeid="10912">iOS 14.0</code>是值。</p>
<p data-nodeid="10708"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9Ed2ACeq0AAKM_xm8xxI584.png" alt="Drawing 5.png" data-nodeid="10916"></p>
<p data-nodeid="10709">有了这些基础知识以后，接下来我就结合 Moments App 来和你介绍下如何进行多环境配置，从而生成不同环境版本的 App。</p>
<h3 data-nodeid="10710">Moments App 构建配置</h3>
<p data-nodeid="10711">一般用 Xcode 编译出不同环境版本的 App 有多种办法，例如拷贝复制所有源代码，建立多个 Target 来包含不同的源码文件等等。不过，在这里我推荐使用 Build Configuration 和 Xcode Scheme 来管理多环境，进而构建出不同环境版本的 App。为什么？因为这两个是目前管理成本最低的办法。接下来我一一介绍下。</p>
<h4 data-nodeid="10712">Build Configuration</h4>
<p data-nodeid="10713">当我们在 Xcode 上新建一个项目的时候，Xcode 会自动生成两个 Configuration：<strong data-nodeid="10934">Debug</strong>和<strong data-nodeid="10935">Release</strong>。Debug 用于日常的本地开发，Release 用于构建和分发 App。而在我们的 Moments App 项目中，有三个 configuration：Debug，Internal 和 AppStore。它们分别用于构建开发环境、测试环境和生产环境。 其中 Internal 和 AppStore 是从自动生成的 Release 拷贝而来的。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/0F/0F/CioPOWA9EfSAY2OBAALUPeNgNGQ665.png" alt="Drawing 7.png" data-nodeid="10933"></p>
<p data-nodeid="10714">那什么是 Build Configuration 呢？</p>
<p data-nodeid="10715"><strong data-nodeid="10941">Build Configuration</strong>就是一组 Build Setting。 我们可以通过 Build Configuration 来分组和管理不同组合的 Build Setting 集合，然后传递给 Xcode 构建系统进行编译。</p>
<p data-nodeid="10716">有了 Build Configuration 以后，我们就能为同一个 Build Setting 设置不同的值。例如<code data-backticks="1" data-nodeid="10943">Build Active Architecture Only</code>在 Debug configuration 是<code data-backticks="1" data-nodeid="10945">Yes</code>，而在 Internal 和 AppStore configuration 则是<code data-backticks="1" data-nodeid="10947">No</code>。这样就能做到同一份源代码通过使用不同的 Build Configuration 来构建出功能不一样的 App 了。</p>
<p data-nodeid="10717"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9EfyAVTM7AAPlUPRPHoQ921.png" alt="Drawing 9.png" data-nodeid="10951"></p>
<p data-nodeid="10718">那么，在构建过程中怎样才能选择不同的 Build Configuration 呢？答案是使用 Xcode Scheme。</p>
<h4 data-nodeid="10719">Xcode Scheme</h4>
<p data-nodeid="10720"><strong data-nodeid="10958">Xcode Scheme</strong>用于定义一个完整的构建过程，其包括指定哪些 Target 需要进行构建，构建过程中使用了哪个 Build Configuration ，以及需要执行哪些测试案例等等。在项目新建的时候只有一个 Scheme，但可以为同一个项目建立多个 Scheme。不过这么多 Scheme 中，同一时刻只能有一个 Scheme 生效。</p>
<p data-nodeid="10721">我们一起看一下 Moments App 项目的 Scheme 吧。 Moments App 项目有三个 Scheme 来分别代表三个环境，Moments Scheme 用于开发环境，Moments-Internal Scheme 用于测试环境，而 Moments-AppStore Scheme 用于生产环境。</p>
<p data-nodeid="10722"><img src="https://s0.lgstatic.com/i/image6/M01/0F/0F/CioPOWA9EgeAYsQhAAy86ZVIPDY023.png" alt="Drawing 11.png" data-nodeid="10962"></p>
<p data-nodeid="10723">下面是<strong data-nodeid="10968">Moments</strong>Scheme 的配置。</p>
<p data-nodeid="10724"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9Eg6AF5o_AA7dS4NRt4E058.png" alt="Drawing 13.png" data-nodeid="10971"></p>
<p data-nodeid="10725">左边是该 Scheme 的各个操作，如当前选择了 Build 操作；右边是对应该操作的配置，比如 Build 对应的 Scheme 可以构建三个不同的 Targets。不同的 Scheme 所构建的 Target 数量可以不一样，例如下面是<strong data-nodeid="10977">Moments-Internal</strong>Scheme 的配置。</p>
<p data-nodeid="10726"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9EheABikxAA6wBftOxsw011.png" alt="Drawing 15.png" data-nodeid="10980"></p>
<p data-nodeid="10727">该 Scheme 只构建主 App Target<strong data-nodeid="10986">Moments</strong>，而不能构建其他两个测试 Target。</p>
<p data-nodeid="10728">当我们选择 Run、Test、Profile、 Analyze 和 Archive 等操作时，在右栏有一个很关键的配置<del data-nodeid="10992">是</del>叫作 Build Configuration，我们可以通过下拉框来选择 Moments App 项目里面三个 Configuration （Debug，Internal 和 AppStore） 中的其中一个。</p>
<p data-nodeid="10729"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9Eh-AcgcJABDw5Uj21A0457.png" alt="Drawing 17.png" data-nodeid="10995"></p>
<p data-nodeid="10730">为了方便管理，我们通常的做法是，一个 Scheme 对应一个 Configuration。有了这三个 Scheme 以后，我们就可以很方便地构建出 Moments α（开发环境），Moments β（测试环境）和 Moments（生产环境）三个功能差异的 App。</p>
<p data-nodeid="10731">￼<br>
<img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9EiqAFjzxAA135y-7y-8462.png" alt="Drawing 19.png" data-nodeid="11001"></p>
<p data-nodeid="10732">你可能已经注意到这三个 App 的名字都不一样，怎么做到的呢？实际上是我们为不同的 Configuration 设置了不一样的 Build Setting。其中决定 App 名字的 Build Setting 叫作<code data-backticks="1" data-nodeid="11003">PRODUCT_BUNDLE_NAME</code>，然后在 Info.plist 文件里面为 Bundle name 赋值，就能构建出名字不一样的 App。</p>
<p data-nodeid="10733"><img src="https://s0.lgstatic.com/i/image6/M01/0F/12/Cgp9HWA9EjyAJgqrAAsJppbAhkc094.png" alt="Drawing 21.png" data-nodeid="11007"></p>
<p data-nodeid="10734">为了构建出不同环境版本的 App，我们需要经常为各个 Build Configuration 下的 Build Setting 设置不一样的值。 在这其中，使用好 xcconfig 配置文件就显得非常重要。</p>
<h3 data-nodeid="10735">xcconfig 配置文件</h3>
<p data-nodeid="10736">xcconfig 会起到什么作用呢？</p>
<p data-nodeid="10737">一般修改 Build Setting 的办法是在 Xcode 的 Build Settings 界面上进行。 例如下面的例子中修改 Suppress Warnings。</p>
<p data-nodeid="10738"><img src="https://s0.lgstatic.com/i/image6/M01/0F/0F/CioPOWA9EkaAcmKDAAScwhg-YKw659.png" alt="Drawing 23.png" data-nodeid="11014"></p>
<p data-nodeid="10739">这样做有一些不好的地方，首先是手工修改很容易出错，例如有时候很难看出来修改的 Setting 到底是 Project 级别的还是 Target 级别的。其次，最关键的是每次修改完毕以后都会修改了 xcodeproj 项目文档 （如下图所示），导致 Git 历史很难查看和对比。</p>
<p data-nodeid="10740"><img src="https://s0.lgstatic.com/i/image6/M01/0F/0F/CioPOWA9ElCAUe6vAAfIFrFWo48879.png" alt="Drawing 25.png" data-nodeid="11018"></p>
<p data-nodeid="10741">幸运的是，Xcode 为我们提供了一个统一管理这些 Build Setting 的便利方法，那就是使用 xcconfig 配置文件来管理。</p>
<h4 data-nodeid="10742">xcconfig 概念及其作用</h4>
<p data-nodeid="10743"><strong data-nodeid="11025">xcconfig</strong>也叫作 Build configuration file（构建配置文件），我们可以使用它来为 Project 或 Target 定义一组 Build Setting。由于它是一个纯文本文件，我们可以使用 Xcode 以外的其他文本编辑器来修改，而且可以保存到 Git 进行统一管理。 这样远比我们在 Xcode 的 Build Settings 界面上手工修改要方便很多，而且还不容易出错。</p>
<p data-nodeid="10744">在 xcconfig 文件里面的每一条 Setting 都是下面的格式：</p>
<pre class="lang-java" data-nodeid="10745"><code data-language="java">BUILD_SETTING_NAME = value
</code></pre>
<p data-nodeid="10746">其中，<code data-backticks="1" data-nodeid="11028">BUILD_SETTING_NAME</code>表示 Build Setting 的名字，而<code data-backticks="1" data-nodeid="11030">value</code>是该 Setting 的值。下面是一个例子。</p>
<pre class="lang-java" data-nodeid="10747"><code data-language="java">SWIFT_VERSION = <span class="hljs-number">5.0</span>
</code></pre>
<p data-nodeid="10748"><code data-backticks="1" data-nodeid="11032">SWIFT_VERSION</code>是用于定义 Swift 语言版本的 Build Setting，其值是<code data-backticks="1" data-nodeid="11034">5.0</code>。Setting 的名字都是由大写字母，数值和下划线组成。这种命名法我们一般成为蛇型命名法，例如<code data-backticks="1" data-nodeid="11036">SNAKE_CASE_NAME</code>。</p>
<p data-nodeid="10749">当我们使用 xcconfig 时，Xcode 构建系统会按照下面的优先级来计算出 Build Setting 的最后生效值：</p>
<ul data-nodeid="10750">
<li data-nodeid="10751">
<p data-nodeid="10752">Platform Defaults (平台默认值)</p>
</li>
<li data-nodeid="10753">
<p data-nodeid="10754">Xcode Project xcconfig File（Project 级别的 xcconfig 文件）</p>
</li>
<li data-nodeid="10755">
<p data-nodeid="10756">Xcode Project File Build Settings（Project 级别的手工配置的 Build Setting）</p>
</li>
<li data-nodeid="10757">
<p data-nodeid="10758">Target xcconfig File （Target 级别的 xcconfig 文件）</p>
</li>
<li data-nodeid="10759">
<p data-nodeid="10760">Target Build Settings（Target 级别的手工配置的 Build Setting）</p>
</li>
</ul>
<p data-nodeid="10761">Xcode 构建系统会按照上述列表从上而下读取 Build Setting，如果发现同样的 Setting ，就会把下面的 Setting 覆盖掉上面的，越往下优先级别越高。</p>
<p data-nodeid="11214" class="te-preview-highlight">例如我们在 Project 级别的 xcconfig 文件配置了<code data-backticks="1" data-nodeid="11216">SWIFT_VERSION = 5.0</code>而在Target 级别的 xcconfig 文件配置了<code data-backticks="1" data-nodeid="11218">SWIFT_VERSION = 5.1</code>，那么Target 级别的 Build Setting 会覆盖 Project 级别的<code data-backticks="1" data-nodeid="11220">SWIFT_VERSION</code>设置，最终<code data-backticks="1" data-nodeid="11222">SWIFT_VERSION</code>生效的值是<code data-backticks="1" data-nodeid="11224">5.1</code>。</p>

<p data-nodeid="10763">那么，要怎样做才能做到不覆盖原有的 Build Setting 呢？我们可以使用下面例子中的<code data-backticks="1" data-nodeid="11057">$(inherited)</code>来实现。</p>
<pre class="lang-java" data-nodeid="10764"><code data-language="java">BUILD_SETTING_NAME = $(inherited) additional value
</code></pre>
<p data-nodeid="10765">可以保留原先的 Setting，然后把新的值添加到后面去。比如：</p>
<pre class="lang-java" data-nodeid="10766"><code data-language="java">FRAMEWORK_SEARCH_PATHS = $(inherited) ./Moments/Pods
</code></pre>
<p data-nodeid="10767">其中的<code data-backticks="1" data-nodeid="11061">FRAMEWORK_SEARCH_PATHS</code>会保留原有的值，然后加上<code data-backticks="1" data-nodeid="11063">./Moments/Pods</code>作为新值。<br>
在配置 Build Setting 时，还可以引用其他已定义的 Build Setting。</p>
<p data-nodeid="10768">例如下面的例子中，<code data-backticks="1" data-nodeid="11068">FRAMEWORK_SEARCH_PATHS</code>使用了另外一个 Build Setting<code data-backticks="1" data-nodeid="11070">PROJECT_DIR</code>。</p>
<pre class="lang-java" data-nodeid="10769"><code data-language="java">FRAMEWORK_SEARCH_PATHS = $(inherited) $(PROJECT_DIR)
</code></pre>
<p data-nodeid="10770">为了重用，我们可以通过<code data-backticks="1" data-nodeid="11073">#include</code>引入其他 xcconfig 文件。</p>
<pre class="lang-java" data-nodeid="10771"><code data-language="java">#include "path/to/OtherFile.xcconfig"
</code></pre>
<h4 data-nodeid="10772">Moments App xcconfig 配置文件</h4>
<p data-nodeid="10773">下面我们一起来看看 Moments App 项目是怎样管理 xcconfig 配置文件吧。</p>
<p data-nodeid="10774"><img src="https://s0.lgstatic.com/i/image6/M00/0F/12/Cgp9HWA9EmOAPIq1ABD7ml3cabY856.png" alt="Drawing 27.png" data-nodeid="11079"></p>
<p data-nodeid="10775">我们把所有 xcconfig 文件分成三大类：Shared、 Project 和 Targets。</p>
<p data-nodeid="10776">其中 Shared 文件夹用于保存分享到整个 App 的 Build Setting，例如 Swift 的版本号、App 所支持的 iOS 版本号等各种共享的基础信息。 下面是 SDKAndDeviceSupport.xcconfig 文件里面所包含的信息：</p>
<pre class="lang-java" data-nodeid="10777"><code data-language="java">TARGETED_DEVICE_FAMILY = <span class="hljs-number">1</span>
IPHONEOS_DEPLOYMENT_TARGET = <span class="hljs-number">14.0</span>
</code></pre>
<p data-nodeid="10778"><code data-backticks="1" data-nodeid="11082">TARGETED_DEVICE_FAMILY</code>表示支持的设备，<code data-backticks="1" data-nodeid="11084">1</code>表示 iPhone。而<code data-backticks="1" data-nodeid="11086">IPHONEOS_DEPLOYMENT_TARGET</code>表示支持 iOS 的最低版本，我们的 Moments App 所支持的最低版本是 iOS 14.0。</p>
<p data-nodeid="10779">Project 文件夹用于保存 Xcode Project 级别的 Build Setting，其中 BaseProject.xcconfig 会引入 Shared 文件夹下所有的 xcconfig 配置文件，如下所示：</p>
<pre class="lang-java" data-nodeid="10780"><code data-language="java">#include "CompilerAndLanguage.xcconfig"
#include "SDKAndDeviceSupport.xcconfig"
#include "BaseConfigurations.xcconfig"
</code></pre>
<p data-nodeid="10781">然后我们会根据三个不同的环境分别建了三个xcconfig 配置文件，如下：</p>
<ul data-nodeid="10782">
<li data-nodeid="10783">
<p data-nodeid="10784">DebugProject.xcconfig 文件</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="10785"><code data-language="java">#include "BaseProject.xcconfig"
SWIFT_ACTIVE_COMPILATION_CONDITIONS = $(inherited) DEBUG
</code></pre>
<ul data-nodeid="10786">
<li data-nodeid="10787">
<p data-nodeid="10788">InternalProject.xcconfig 文件</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="10789"><code data-language="java">#include "BaseProject.xcconfig"
SWIFT_ACTIVE_COMPILATION_CONDITIONS = $(inherited) INTERNAL
</code></pre>
<ul data-nodeid="10790">
<li data-nodeid="10791">
<p data-nodeid="10792">AppStoreProject.xcconfig 文件</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="10793"><code data-language="java">#include "BaseProject.xcconfig"
SWIFT_ACTIVE_COMPILATION_CONDITIONS = $(inherited) PRODUCTION
</code></pre>
<p data-nodeid="10794">它们的共同点是都引入了用于共享的 BaseProject.xcconfig 文件，然后分别定义了 Swift 编译条件配置<code data-backticks="1" data-nodeid="11094">SWIFT_ACTIVE_COMPILATION_CONDITIONS</code>。其中<code data-backticks="1" data-nodeid="11096">$(inherited)</code>表示继承原有的配置，<code data-backticks="1" data-nodeid="11098">$(inherited)</code>后面的<code data-backticks="1" data-nodeid="11100">DEBUG</code>或者<code data-backticks="1" data-nodeid="11102">INTERNAL</code>表示在原有配置的基础上后面添加了一个新条件。有了这些编译条件，我们就可以在代码中这样使用：</p>
<pre class="lang-java" data-nodeid="10795"><code data-language="java">#if DEBUG
    print("Debug Environment")
#endif
</code></pre>
<p data-nodeid="10796">该段代码只在开发环境执行，因为只有开发环境的<code data-backticks="1" data-nodeid="11105">SWIFT_ACTIVE_COMPILATION_CONDITIONS</code>才有<code data-backticks="1" data-nodeid="11107">DEBUG</code>的定义。这样做能有效分离各个环境，保证同一份代码构建出对应不同环境的 App。</p>
<p data-nodeid="10797">Targets 文件夹用于保存 Xcode Target 级别的 Build Setting，也是由一个 BaseTarget.xcconfig 文件来共享所有 Target 都需要使用的信息。</p>
<pre class="lang-java" data-nodeid="10798"><code data-language="java">PRODUCT_BUNDLE_NAME = Moments
</code></pre>
<p data-nodeid="10799">这里的<code data-backticks="1" data-nodeid="11111">PRODUCT_BUNDLE_NAME</code>是 App 的名字。<br>
下面是三个不同环境的 Target xcconfig 文件。</p>
<ul data-nodeid="10800">
<li data-nodeid="10801">
<p data-nodeid="10802">DebugTarget.xcconfig</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="10803"><code data-language="java">#include "../Pods/Target Support Files/Pods-Moments/Pods-Moments.debug.xcconfig"
#include "BaseTarget.xcconfig"
PRODUCT_BUNDLE_NAME = $(inherited) α
PRODUCT_BUNDLE_IDENTIFIER = com.ibanimatable.moments.development
</code></pre>
<ul data-nodeid="10804">
<li data-nodeid="10805">
<p data-nodeid="10806">InternalTarget.xcconfig</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="10807"><code data-language="java">#include "../Pods/Target Support Files/Pods-Moments/Pods-Moments.internal.xcconfig"
#include "BaseTarget.xcconfig"
PRODUCT_BUNDLE_NAME = $(inherited) β
PRODUCT_BUNDLE_IDENTIFIER = com.ibanimatable.moments.internal
</code></pre>
<ul data-nodeid="10808">
<li data-nodeid="10809">
<p data-nodeid="10810">AppStoreTarget.xcconfig</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="10811"><code data-language="java">#include "../Pods/Target Support Files/Pods-Moments/Pods-Moments.appstore.xcconfig"
#include "BaseTarget.xcconfig"
PRODUCT_BUNDLE_NAME = $(inherited)
PRODUCT_BUNDLE_IDENTIFIER = com.ibanimatable.moments
</code></pre>
<p data-nodeid="10812">它们都需要引入 CocoaPods 所生成的 xcconfig 和共享的 BaseTarget.xcconfig 文件，然后根据需要改写 App 的名字。例如<strong data-nodeid="11131">Debug</strong>Target 覆盖了<code data-backticks="1" data-nodeid="11123">PRODUCT_BUNDLE_NAME</code>的值为<code data-backticks="1" data-nodeid="11125">Moments α*</code>, 其所构建的 App 叫作<strong data-nodeid="11132">Moments α</strong>。</p>
<p data-nodeid="10813">一般在 App Store 上所有 App 的标识符都必须是唯一的。如果你的项目通过 Configuration 和 Scheme 来生成免费版和收费版的 App，那么，你必须在两个 Configuration 中分别为<code data-backticks="1" data-nodeid="11134">PRODUCT_BUNDLE_IDENTIFIER</code>配置对应的标识符，例如<code data-backticks="1" data-nodeid="11136">com.lagou.free</code>和<code data-backticks="1" data-nodeid="11138">com.lagou.paid</code>。</p>
<p data-nodeid="10814">在 Moments App 中，我们也为各个环境下的 App 使用了不同的标识符，以方便我们通过 CI 自动构建，并分发到内部测试组或者 App Store。同时，这也能为各个环境版本的 App 分离用户行为数据，方便统计分析。</p>
<p data-nodeid="10815">一旦有了这些 xcconfig 配置文件，今后我们就可以在 Xcode 的 Project Info 页面里的 Configurations 上引用它们。</p>
<p data-nodeid="10816"><img src="https://s0.lgstatic.com/i/image6/M00/0F/12/Cgp9HWA9EnyAOX4GABH86gYqfAc683.png" alt="Drawing 29.png" data-nodeid="11144"></p>
<p data-nodeid="10817">下面是所有 Configurations 所引用的 xcconfig 文件。</p>
<p data-nodeid="10818"><img src="https://s0.lgstatic.com/i/image6/M00/0F/12/Cgp9HWA9EoSARVwGAAZ4k9BLebE007.png" alt="Drawing 31.png" data-nodeid="11148"></p>
<p data-nodeid="10819">在配置好所有 xcconfig 文件的引用以后，可以在 Build Settings 页面查看某个 Build Setting 的生效值。我们以<code data-backticks="1" data-nodeid="11150">IPHONEOS_DEPLOYMENT_TARGET</code>为例，一起看看。</p>
<p data-nodeid="10820"><img src="https://s0.lgstatic.com/i/image6/M00/0F/0F/CioPOWA9EoyAfB15AAMBWeDhWeI967.png" alt="Drawing 33.png" data-nodeid="11154"></p>
<p data-nodeid="10821">当我们选择<strong data-nodeid="11164">All</strong>和<strong data-nodeid="11165">Levels</strong>时，可以看到所有配置信息分成了不同的列。这些列分别代表前面的 Build Settng 优先级：</p>
<ul data-nodeid="10822">
<li data-nodeid="10823">
<p data-nodeid="10824">平台默认值</p>
</li>
<li data-nodeid="10825">
<p data-nodeid="10826">Project 级别的 xcconfig 文件</p>
</li>
<li data-nodeid="10827">
<p data-nodeid="10828">Xcode 项目文件中的 Project 级别配置</p>
</li>
<li data-nodeid="10829">
<p data-nodeid="10830">Target 级别的 xcconfig 文件</p>
</li>
<li data-nodeid="10831">
<p data-nodeid="10832">Xcode 项目文件中的 Target 级别配置</p>
</li>
</ul>
<p data-nodeid="10833">Build Settng 的优先级是从左到右排序的。越是左边优先级就越高。例如，我们在 Project 级别的 xcconfig 文件里面定义了<code data-backticks="1" data-nodeid="11172">IPHONEOS_DEPLOYMENT_TARGET</code>的值为<code data-backticks="1" data-nodeid="11174">14.0</code>，那么Project 级别的 xcconfig 文件（Project Config File） 一列上就会显示<code data-backticks="1" data-nodeid="11176">iOS 14.0</code>，它覆盖了系统的默认值 （iOS Default）<code data-backticks="1" data-nodeid="11178">iOS 14.2</code>。这就是因为 Project 级别的 xcconfig 文件，它的优先级高于系统默认值，因此最后生效的值是<code data-backticks="1" data-nodeid="11180">iOS 14.0</code>。</p>
<h3 data-nodeid="10834">总结</h3>
<p data-nodeid="10835">本讲我介绍了如何通过 Build Configuration、 Xcode Scheme 以及 xcconfig 配置文件来统一项目的构建配置，从而搭建出多个不同环境，为后期构建出对应环境的 App 做准备。</p>
<p data-nodeid="10836"><img src="https://s0.lgstatic.com/i/image6/M00/0F/12/Cgp9HWA9EpaASYdnAAaN1YenVoI219.png" alt="Drawing 34.png" data-nodeid="11186"></p>
<p data-nodeid="10837">在使用 xcconfig 配置时，还是需要注意以下两点：</p>
<p data-nodeid="10838">首先，我们必须把所有 Build Setting 都配置在 xcconfig 文件里面，并通过 Git 进行统一管理；</p>
<p data-nodeid="10839">其次，我们千万不要在 Xcode 的 Build Settings 页面修改任何 Setting，否则该配置会覆盖 xcconfig 文件里面的配置。如果你不小心修改了，可以通过点击删除键把页面是的配置删掉。</p>
<p data-nodeid="10840">思考题：</p>
<blockquote data-nodeid="10841">
<p data-nodeid="10842">请问我们 Moments App 项目的主 App 为什么只使用了一个 Target 吗？如果使用多个 Target，例如 Debug Target，Internal Target 和 Release Target 会有什么问题？</p>
</blockquote>
<p data-nodeid="10843">你可以把回答写到下面的留言区哦，下一讲我将介绍如何使用 Swiftlint 统一编码规范。</p>
<p data-nodeid="10844"><strong data-nodeid="11196">源码地址：</strong></p>
<blockquote data-nodeid="10845">
<p data-nodeid="10846"><a href="https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Configurations" data-nodeid="11199">https://github.com/lagoueduCol/iOS-linyongjian/tree/main/Moments/Moments/Configurations</a></p>
</blockquote>
<hr data-nodeid="10847">
<p data-nodeid="10848"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="11204"><img src="https://s0.lgstatic.com/i/image6/M00/08/77/Cgp9HWA0wqWAI70NAAdqMM6w3z0673.png" alt="Drawing 1.png" data-nodeid="11203"></a></p>
<p data-nodeid="10849"><strong data-nodeid="11208">《大前端高薪训练营》</strong></p>
<p data-nodeid="10850" class="">12 个月打磨，6 个月训练，优秀学员大厂内推，<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="11212">点击报名，高薪有你</a>！</p>

---

### 精选评论

##### *孟：
> 多个 Target 维护增加了成本：### 1. 新建类和资源文件添加时需要逐个勾选相应的 Target \n ### 2. 如果疏漏、会造成不同 Target 的功能缺失，Bug 数量增加 \n ### 3. 维护起来和开发起来都增加了成本

##### *三：
> https://xcodebuildsettings.com/在这里找buildsettings对应的xcconfig中的key

##### *虎：
> 思考题考查的是啥？是想通过多个 target 代替多个 Scheme 吗？还是target 有何意义？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦哦，我出这一题的原因是在我们以前的项目都是使用多个 Target 而不是多个 Scheme 和 Build Configuration 来配置多环境的，如果使用多个 Target 的情况下，每次新增一个源代码文件的时候都需要在 Xcode 右边的 File inspector 里的 Target Membership 上同时选择多个 Target。我为了大家不要踩这个坑而出了这样一题。

##### **3382：
> 老师，你好 麻烦问一下 如果两个客户端有很多相似的业务逻辑，比如一个教育类型APP有老师端和家长端，除了常用的工具类之外，这两个端有很多一样的界面，比如聊天界面，视屏界面都一样，能采取这种方法来实现代码的复用吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个要看具体需求呀，如果我来架构你说的情况，一般把各个功能模块放到独立的 Pod 里面，然后给不同端的 App 来引用。不同 Target 比较适合用于隐藏一些功能，例如收费 App 和免费 App。同一个 App 换不同的主题颜色等等。总的来说不同 Target 的 App 的相似度非常高。

##### *利：
> "例如我们在Project 级别的 xcconfig 文件配置了SWIFT_VERSION = 5.0而在Target 级别的 xcconfig 文件配置了SWIFT_VERSION = 5.1，那么Target 级别的 Build Setting也就是Target 级别的例子中的5.1会覆盖 Project 级别的SWIFT_VERSION设置，最终SWIFT_VERSION生效的值是5.0。"这个最后结论应该是：最终SWIFT_VERSION生效的值是5.1吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你是对的，是 5.1，谢谢指正~已经修改了

##### **9176：
> 只用一个Target 和 Scheme, 只是出包时选择不同的 configuration 不就够了吗？没必要出多个scheme吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Scheme 能帮助我们很方便地打出不同版本的 App，请参考 《第 25 讲 | 自动化构建：解决大量重复性人力工作神器》怎样打包和签名。

##### **龙：
> 林老师，请教个问题，我有看到在Target.xcconfig相关文件中按环境区分了PRODUCT_BUNDLE_IDENTIFIER，但是有些三方的SDK若是需要绑定bundleI Identifier的话，这里有比较好的处理方案么？

##### **松：
> Unable to load contents of file list: '/Target Support Files/Pods-Moments/Pods-Moments-frameworks-Debug-output-files.xcfilelist'老师，我自己创建了一个工程，运行时报错了，不知道是什么导致的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你在那里创建一个新工程？为什么新工程会使用 Pods-Moments 的文件呢？

##### *杨：
> 老师你好 请问每个组件库也都需要配置一遍这个环境吗?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不需要呀，假如你使用 Pod，编译的时候还是在你的主 App 上的。

##### **一：
> 我们现在用 XcodeGen 来生成 .xcworkspace 不使用 Git 管理。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个真没用过呢，每次打包都重新 Gen 一次 .xcworkspace 文件吗？这样做有什么好处吗？能否分享一下经验？

##### **昆：
> 如何知道，build setting 中对应的BUILD_SETTING_NAME 呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以到这个网站 https://xcodebuildsettings.com/ 查找

##### **泽：
> 我反复对比了自己的工程和老师的demo，差别就是bundle identifier的值不一样，老师的demo配置的值显示Multiple Values，我的demo显示的就是创建工程的bundle ID，我把project.pbxproj文件里的PRODUCT_BUNDLE_IDENTIFIER都删了， 就显示Multiple Values，之后再改xcconfig文件就生效了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，我强调过要把 Xcode 配置的值通过删除键删掉，只使用 xcconfig 的配置。

##### **泽：
> 在info.plist里， 这个$(PRODUCT_BUNDLE_IDENTIFIER) 是默认引用的， 改config配置文件就不生效。老师您是直接在config文件定义好PRODUCT_BUNDLE_IDENTIFIER就起作用了吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我不是很明白为什么不生效，请参考原文里面的这一段“在配置好所有 xcconfig 文件的引用以后，可以在 Build Settings 页面查看某个 Build Setting 的生效值。我们以 IPHONEOS_DEPLOYMENT_TARGET 为例，一起看看。 ”

建议你 把 "PRODUCT_BUNDLE_IDENTIFIER" 放进搜索框，Xcode 手工配置的优先级比 xcconfig 文件的高。是不是你在 Xcode 里手工配置了这个值？如果是，需要点击 “delete” 键删掉。

文件中也强调过这一点，请看看原文的这一部分“其次，我们千万不要在 Xcode 的 Build Settings 页面修改任何 Setting，否则该配置会覆盖 xcconfig 文件里面的配置。如果你不小心修改了，可以通过点击删除键把页面是的配置删掉。”

如果还有问题，麻烦再回复。

##### **泽：
> PRODUCT_BUNDLE_IDENTIFIER老师， 配置这个不生效，网上查说这个配置不生效注意：有部分变量不能通过xcconfig配置到Build Settings，比如 配置PRODUCT_BUNDLE_IDENTIFIER不会起作用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个配置需要在 plist 文件里面，如果打开 info.plist 能看到如何引用该 Build Setting。

##### **泽：
> 是的， 我在config文件里配置了PRODUCT_BUNDLE_NAME，配置完它就在User-Defined有这个值了， 需要修改info.plist的Bundle name这值成为$(PRODUCT_BUNDLE_NAME)，这块是需要手动改的吧，这是我疑惑的点，老师。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的呀，需要在 info.plist 里面指向 PRODUCT_BUNDLE_NAME，这样做的好处是可以使得同一个 info.plist 文件能从不同的 Schema 来获取不同的值。这比使用多个不同的 info.plist 文件更容易管理。可以打开 info.plist 来看看。配置如下 <key>CFBundleName</key><string>$(PRODUCT_BUNDLE_NAME)</string>

##### **泽：
> 老师，比如SWIFT_VERSION和IPHONEOS_DEPLOYMENT_TARGET在BuildSetting里有对应的Key，在配置文件里配置就可以，PRODUCT_BUNDLE_NAME是给infoPlist用的值，它是在BuildSetting没有的，所以在User-Defined里生成，需要手动配置到InfoPlist里吧才可以生效。 是这么个意思吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，我明白你的意思了，PRODUCT_BUNDLE_NAME 只是用在 info.plist 里面，不需要在 Xcode 的配置页面进行配置了，只需要在 xcconfig 里面配置好即可。

##### **泽：
> 配置文件的值在User-Defined也能体现出来，但是xcode的选项就是不生效， 老师是否有交流方式呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我不是很明白你说的情况，你说自己在 xcconfig 定义的 Build Settings 不生效吗？可以以  PRODUCT_BUNDLE_IDENTIFIER 作为例子，它在几个 xcconfig 和 info.plist 都用到呀。

##### **泽：
> 老师，我配置好了， 但是不生效， 比如用xcconfig文件配置了系统的版本，app名字， bundleID等，就不需要手动去设置xcode选项了吧， 但是我配置完了，还是没生效，依旧是14.0，

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 配置完 xcconfig 文件以后就不需要在 Xcode 里面手工配置了，请根据文章里面的步骤看看最终生效值。在这一段有描述 “在配置好所有 xcconfig 文件的引用以后，可以在 Build Settings 页面查看某个 Build Setting 的生效值。我们以 IPHONEOS_DEPLOYMENT_TARGET 为例，一起看看。 ” Resovled 是最终生效值。从左往右看就能看到值来自哪里。

##### **亮：
> 1. Project下面的xcconfig和Target下面的xcconfig是否需要在Target Membership中勾连target？因为命名上看起来以为Target下面的xcconfig创建的时候需要跟Target关联起来。2. 新创建xcconfig时，如果使用了真实的Project、Target、Share目录，直接#include会报错，需要注意一下。为什么会使用#include？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 所有 xcconfig 文件都不需要在 Target Membership 中勾连 target。 在文章讲述了“一旦有了这些 xcconfig 配置文件，今后我们就可以在 Xcode 的 Project Info 页面里的 Configurations 上引用它们。”并提供了一张截图，可以看看。
2. 直接 #include会报错 是报什么错？是路径不对吗？请模仿 Moments App 的路径来存放你的 xcconfig 文件。

##### **6158：
> 请老师帮忙看一下我对于target和scheme的理解是否正确，我理解的是target侧重于不同功能，一个target有的功能另一个target不一定有，scheme侧重于用不同settings完成相同功能，一个scheme可以设置prints out所有的logs，另一个scheme设置成不打印任何log，但是不论选哪个scheme功能都是相同的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我觉得你的理解是大致是对的，我们可以用不同的 Target 来做不同的功能，例如 Apple Watch Target 做手表 App，Widget Target 做屏幕 Widget。我们使用 Scheme 来引用不同的 Build Configurations 把环境进行分离，从而方便测试与验证。打印 Log 是其中的一种效果。

##### *洋：
> 希望更新快点

##### **辉：
> 一口气读完了前三篇文章，有机会实践下加深理解，最后期待老师的更新~

##### **5627：
> 老师，请教一下，如果PRODUCT_BUNDLE_NAME 国际化不同语言时要怎么配置呢，谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果是为了 App 在不同语言的设备上显示不同的名称，可以新增一个 InfoPlist.strings 文件呀，然后在 Info.plist 里面填入你在 InfoPlist.strings 里面配置的 Key，系统就可以自动为各个语言生成不同的 App 名称。

##### **坚：
> 感谢，收货很多。之前都是手动去修改的，而且也不是很理解project，target，scheme之间的关系。

##### *腾：
> 打卡

##### *召：
> 说说自己的拙见…多个target，不利于维护，一个target修改，其他的target也要跟着修改，代码的也没有复用性可言😅

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦哦，请问能不能举一些具体的例子？我们的项目还没有碰到你说的问题呢。

##### *勇：
> 图标，URLScheme可以通过这个方式也配置成不一样吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的呀，也是通过 Build Configuration 来做。首先要在 Asset Catalog 加上新的 Icon set，例如命名为 AppIcon.beta  （你可以在原先的 Icon 图标的基础上加一个 Beta 的 Label），然后在不同的 xcconfig 里面配置 ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon.beta 等值。我建议你在 Moments App 做一遍并提交一个 PR，对你理解各种构建配置会有很多帮助。

##### **辉：
> 干货满满，就是更新太慢了

##### *哥：
> 留个言

