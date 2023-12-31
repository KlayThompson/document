### 本资源由 itjc8.com 收集整理
<p data-nodeid="849" class="">在 iOS 开发过程中，你是不是会经常遇到这些情况：</p>
<p data-nodeid="850">每次打开一个新项目，都需要手动搭建开发环境；有时候在安装第三方工具时使用到 sudo 权限，导致以后安装工具都需要手工输入密码而无法实施自动化。还有，每当启动一台新 CI 时，就需要手工登录并配置一遍，更可怕的是，原先搭建好的 CI 会随着 Xcode 版本更新需要重新配置。</p>
<p data-nodeid="851">为什么会这么麻烦呢？就是因为你在项目开始之初没有做好统一配置。</p>
<p data-nodeid="852">所谓统一配置，就是所有的配置信息都以文本的格式存放在 Git 里面，我们可以随时查看修改记录，以此来帮助我们比较不同配置之间的差异性，然后在这个基础上不断更新迭代。</p>
<p data-nodeid="853">可以说，有了统一配置，任何工程师都可以搭建出一模一样的开发环境，构建出功能一致的 App；有了统一配置，还可以让我们按需延展 CI 服务，而不用任何手工操作。更重要的是，它还可以应用到各个类似的 iOS 项目中，极大地减轻了项目前期的搭建成本。</p>
<p data-nodeid="854">既然统一的配置那么重要，那么我们怎样搭建统一配置的开发环境呢？</p>
<h3 data-nodeid="855">Ruby 工具链</h3>
<p data-nodeid="856">我们可以通过 Ruby 工具链为整个项目搭建一致的开发和构建环境。为什么选择 Ruby 而不是其他语言环境呢？因为在 iOS 开发方面，目前流行的第三方工具 CocoaPods 和 fastlane 都是使用 Ruby 来开发的。特别是 Ruby 有非常成熟的依赖库管理工具 RubyGems 和 Bundler，其中 Bundler 可以帮我们有效地管理 CocoaPods 和 fastlane 的版本。</p>
<p data-nodeid="857">下面一起来看看怎样搭建一个统一的开发环境吧。</p>
<p data-nodeid="858"><img src="https://s0.lgstatic.com/i/image6/M01/0A/3D/Cgp9HWA3F9SAPGu_AALEFvdRBzw539.png" alt="图片2.png" data-nodeid="924"></p>
<div data-nodeid="859"><p style="text-align:center">开发环境统一配置图</p></div>
<p data-nodeid="860">通常，统一的开发环境应该从操作系统开始。对于 iOS 开发来说，<strong data-nodeid="930">MacOS</strong> 是目前 iOS 开发唯一支持的操作系统。在公司，MacOS 的版本一般由 IT 部门统一管理和更新。要注意，当公司统一更新了我们开发环境的 MacOS 版本以后，需要同时更新 CI 上 MacOS 的版本，以保持一致。</p>
<h4 data-nodeid="861">Xcode</h4>
<p data-nodeid="862">位于 MacOS 上层的是 Xcode 和 rbenv。其中，<strong data-nodeid="937">Xcode</strong> 是 iOS 开发和构建工具，在同一个项目里，最好使用同一个版本的 Xcode 进行开发和构建，我们可以在项目的 README.md 文件标注 Xcode 的版本。</p>
<p data-nodeid="863">像我们将要开发的这款类似朋友圈的 Moments App 项目，我就在对应的 README.md 文件里标明了需要使用 Xcode Version 12.2 (12B45b)。具体内容你也可以在<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/README.md" data-nodeid="941">代码仓库</a>找到。</p>
<p data-nodeid="864"><img src="https://s0.lgstatic.com/i/image6/M01/0A/3D/Cgp9HWA3GA-AYvZfAAND2HpeMGs775.png" alt="图片3.png" data-nodeid="945"></p>
<p data-nodeid="865">那我们怎样才能保证每个人都安装同一个版本号的 Xcode 呢？技巧就是我们不要到有自动更新功能的 Mac App Store 中下载 Xcode，而是到<a href="https://developer.apple.com/download/more/" data-nodeid="949">苹果的开发者网站</a>搜索并下载。</p>
<p data-nodeid="866"><img src="https://s0.lgstatic.com/i/image6/M00/0A/39/CioPOWA3F5OAUokNAARwNdiYhbs294.png" alt="图片1.png" data-nodeid="953"></p>
<p data-nodeid="867">有时候我们会同时开发多个项目，这样有可能要安装多个不同版本的 Xcode。如果你的机器有多于一个版本的 Xcode，此时需要特别注意，为了保证所使用的编译器版本一致，在每次执行自动化命令之前（如执行<code data-backticks="1" data-nodeid="955">bundle exec fastlane test</code>），要先使用<code data-backticks="1" data-nodeid="957">xcode-select -s</code>来选择该项目所对应版本的 Xcode。</p>
<p data-nodeid="868">比如说我的电脑上有多个 Xcode 版本，在开发 Moments App 时，每次执行自动化命令之前都会执行这样一条命令<code data-backticks="1" data-nodeid="960">xcode-select -s /Applications/Xcode12.2.app/Contents/Developer</code>来选择 Moments App 项目所使用的 Xcode。这里的<code data-backticks="1" data-nodeid="962">Xcode12.2.app</code>就是我安装的 Xcode 12.2 版所在的位置。</p>
<h4 data-nodeid="869">rbenv</h4>
<p data-nodeid="870">有了版本一致的 Xcode 以后，因为后期我们会用到 CocoaPods 等第三方 Ruby 工具，为了自动化安装和管理这些工具，整个项目团队所使用的 Ruby 版本也必须保持一致。为此，我们就需要用到 Ruby 环境管理工具。</p>
<p data-nodeid="871">目前流行的 Ruby 环境管理工具有 RVM 和 rbenv。我推荐使用的是 rbenv，因为它使用 shims 文件夹来分离各个 Ruby 版本，相对于 RVM 更加轻装而方便使用。千万注意，团队内部不要同时使用不同的 Ruby 环境管理工具，否则项目编译会出错。</p>
<p data-nodeid="872"><strong data-nodeid="971">rbenv</strong> 是 Ruby 环境管理工具，能够安装、管理、隔离以及在多个 Ruby 版本之间切换。要使用 rbenv，我们可以通过 Homebrew 来安装它，下面是安装 Homebrew 和 rbenv 的脚本。</p>
<pre class="lang-java" data-nodeid="1699"><code data-language="java">$ /bin/bash -c <span class="hljs-string">"$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"</span>
brew install rbenv ruby-build rbenv-vars
</code></pre>
<p data-nodeid="2600" class="">一旦安装 rbenv 完毕，我们需要把以下的设置信息放到你的 Shell 配置文件里面，例如 ~/.bash_profile 或者 ~/.zshrc 等文件，这样能保证每次打开终端的时候都会初始化 rbenv。</p>
<pre class="lang-java te-preview-highlight" data-nodeid="9877"><code data-language="java">export PATH=<span class="hljs-string">"$HOME/.rbenv/bin:$PATH"</span> 
eval <span class="hljs-string">"$(rbenv init -)"</span>
</code></pre>



<p data-nodeid="8553" class="">接着我们就可以安装和设置项目的 Ruby 环境了。</p>























<pre class="lang-java" data-nodeid="875"><code data-language="java">$ cd $(PROJECT_DIR)
$ rbenv install <span class="hljs-number">2.7</span><span class="hljs-number">.1</span>
$ rbenv local <span class="hljs-number">2.7</span><span class="hljs-number">.1</span>
</code></pre>
<p data-nodeid="876">此处是把项目的 Ruby 环境配置为 2.7.1 版本。rbenv 会帮我们建立 一个叫作.<strong data-nodeid="982">ruby-version</strong> 的文件，该文件里面只保存一个版本号（例如<code data-backticks="1" data-nodeid="978">2.7.1</code>）的字符串。这个包含了版本号的文件可以用 Git 进行管理。如果要更新版本，可以通过<code data-backticks="1" data-nodeid="980">rbenv local</code>命令进行，每次更新也由 Git 统一管理，这样就能让其他开发者使用同一版本的 Ruby 开发环境了。</p>
<h4 data-nodeid="877">RubyGems 和 Bundler</h4>
<p data-nodeid="878">RubyGems 和 Bundler 主要是用来安装和管理 CocoaPods 和 fastlane 等第三方工具。</p>
<p data-nodeid="879">具体来说，RubyGems&nbsp;是 Ruby 依赖包管理工具。在 Ruby 的世界，包叫作 Gem，我们可以通过<code data-backticks="1" data-nodeid="986">gem install</code>命令来安装。但是 RubyGems 在管理 Gem 版本的时候有些缺陷，就有人开发了 Bundler，用它来检查和安装 Gem 的特定版本，以此为 Ruby 项目提供一致性的环境。</p>
<p data-nodeid="880">要安装 Bundler，我们可执行<code data-backticks="1" data-nodeid="989">gem install bundler</code>命令进行，之后，再执行<code data-backticks="1" data-nodeid="991">bundle init</code>就可以生成一个 Gemfile 文件，像 CocoaPods 和 fastlane 等依赖包，我们就可以添加到这个文件里面。</p>
<p data-nodeid="881">具体代码如下：</p>
<pre class="lang-java" data-nodeid="882"><code data-language="java">source <span class="hljs-string">"https://rubygems.org"</span>
gem <span class="hljs-string">"cocoapods"</span>, <span class="hljs-string">"1.10.0"</span>
gem <span class="hljs-string">"fastlane"</span>, <span class="hljs-string">"2.166.0"</span>
</code></pre>
<p data-nodeid="883">注意我们在<code data-backticks="1" data-nodeid="995">gem</code>命令里面都指定了依赖包的特定版本号。例如，在我们的 Moment App 就使用了<code data-backticks="1" data-nodeid="997">1.10.0</code>版的 CocoaPods，然后执行<code data-backticks="1" data-nodeid="999">bundle install</code>来安装各个 Gem。 Bundler 会自动生成一个 Gemfile.lock 文件来锁定所安装的 Gem 的版本，例如：</p>
<pre class="lang-java" data-nodeid="884"><code data-language="java"><span class="hljs-function">DEPENDENCIES
  <span class="hljs-title">cocoapods</span> <span class="hljs-params">(= <span class="hljs-number">1.10</span><span class="hljs-number">.0</span>)</span>
  <span class="hljs-title">fastlane</span> <span class="hljs-params">(= <span class="hljs-number">2.166</span><span class="hljs-number">.0</span>)</span>
</span></code></pre>
<p data-nodeid="885">为了保证团队其他成员都可以使用版本号一致的 Gem，我们需要把 Gemfile 和 Gemfile.lock 一同保存到 Git 里面统一管理起来。</p>
<p data-nodeid="886">到此为止，我们已经知道怎样使用 Ruby 工具链配置一个统一的开发环境。但在真实的开发环境中，搭建环境只需要一个人来完成即可，其他成员执行以下脚本就能完成整套开发环境的搭建。</p>
<pre class="lang-java" data-nodeid="887"><code data-language="java">$ ./scripts/setup.sh
</code></pre>
<p data-nodeid="888">我们一起看看这个脚本做了些什么？</p>
<pre class="lang-java" data-nodeid="889"><code data-language="java"># Install ruby using rbenv
ruby_version=`cat .ruby-version`
if [[ ! -d "$HOME/.rbenv/versions/$ruby_version" ]]; then
  rbenv install $ruby_version;
fi
# Install bunlder
gem install bundler
# Install all gems
bundle install
# Install all pods
bundle exec pod install
</code></pre>
<p data-nodeid="890">该脚本主要做了四件事情，第一步是在 rbenv 下安装特定版本的 Ruby 开发环境，然后通过 RubyGems 安装 Bunlder，接着使用 Bundler 安装 CocoaPods 和 fastlane 等依赖包，最后安装各个 Pod。这样，一个统一的项目环境就搭建完成了，接下来开发者就可以打开 <strong data-nodeid="1009">Moments.xcworkspace</strong>进行开发了。</p>
<p data-nodeid="891">说完 Ruby 环境搭建以后，最后我们一起聊聊保证项目文件一致性的 .gitignore 文件。</p>
<h3 data-nodeid="892">.gitignore 文件</h3>
<p data-nodeid="893">.gitignore 文件是一个配置文件，用来指定让 Git 需要忽略的文件或者目录。如果没有 .gitignore 文件，项目成员可能会不小心把一些自动生成等无关重要的文件或者具有个人信息(例如 xcuserdata)的文件保存到 Git 里面。这就大大增加了查看 Git 修改历史的难度。因此，在项目初期就配置一个合适的 .gitignore 文件，能减轻后续的管理工作。</p>
<p data-nodeid="894">如何创建 .gitignore 文件呢？</p>
<p data-nodeid="895">我一般会在 gitignore.io 里面输入关键字，例如 Xcode，Swift 等，然后该网站会帮我们生成一个默认的 .gitignore 文件。咱们项目 Moments App 的.gitignore 文件你可以到<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.gitignore" data-nodeid="1017">拉勾教育的仓库中</a>查看。</p>
<p data-nodeid="896"><img src="https://s0.lgstatic.com/i/image6/M01/08/75/CioPOWA0xDCALL3FAABG42Ok7aU292.png" alt="Drawing 3.png" data-nodeid="1021"></p>
<h3 data-nodeid="897">总结</h3>
<p data-nodeid="898">以上，我们通过 Xcode、rbenv、RubyGems 和 Bundler 搭建一个统一的 iOS 开发和构建环境。</p>
<p data-nodeid="899"><img src="https://s0.lgstatic.com/i/image6/M01/0A/39/CioPOWA3FsqAOn0YAAq1IyqbxEs043.png" alt="开发环境.png" data-nodeid="1026"></p>
<p data-nodeid="900">再次强调下，为了让各个开发和构建环境能保持一致，我们要把 .ruby-version、 Gemfile 和 Gemfile.lock 文件通过 Git 统一管理起来，并共享给整个项目团队使用。</p>
<p data-nodeid="901">而且，由于我们的开发环境已经通过 Bundler 管理起来，今后，当使用各个 Gem 工具的时候，也需要使用 Bundler。例如在使用 CocoaPods 时要执行<code data-backticks="1" data-nodeid="1029">bundle exec pod</code>，以保证我们使用的是项目级别而不是操作系统级别的 Gem 工具。</p>
<p data-nodeid="902">思考题：</p>
<blockquote data-nodeid="903">
<p data-nodeid="904">请问如果我们不使用 rbenv ，那我们使用的 Ruby 来自哪里？使用 CocoaPods 等工具又来自哪里？不同项目能使用不同版本的 CocoaPods 吗？</p>
</blockquote>
<p data-nodeid="905">你可以把回答写到下面的留言区哦，下一讲我将介绍如何使用 CocoaPods 统一依赖库的管理。</p>
<p data-nodeid="906">源码地址：</p>
<blockquote data-nodeid="907">
<p data-nodeid="908">README.md<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/README.md" data-nodeid="1039">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/README.md</a><br>
Moments App 的.gitignore 文件<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.gitignore" data-nodeid="1045">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.gitignore</a></p>
</blockquote>
<hr data-nodeid="909">
<p data-nodeid="910"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="1050"><img src="https://s0.lgstatic.com/i/image6/M00/08/77/Cgp9HWA0wqWAI70NAAdqMM6w3z0673.png" alt="Drawing 1.png" data-nodeid="1049"></a></p>
<p data-nodeid="911"><strong data-nodeid="1054">《大前端高薪训练营》</strong></p>
<p data-nodeid="912" class="">12 个月打磨，6 个月训练，优秀学员大厂内推，<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="1058">点击报名，高薪有你</a>！</p>

---

### 精选评论

##### *虎：
> “例如在使用 CocoaPods 时要执行bundle exec pod，以保证我们使用的是项目级别而不是操作系统级别的 Gem 工具”，如何保证团队的其他开发人员使用bundle exec pod 而不是操作系统级别的 Gem 工具呢?只能通过文档的形式约束吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是一个很好问题呀，对于 bundle exec 的使用还是以教育为主，我们也可以把常用的脚步放在一个 Shell Script 里面。例如 Moments App 里面的 setup.sh。不知道你在这方面有没有更好的办法呢？

##### **文：
> 这里配置rbenv，好像少了PATH 配置在~/.bash_profile 增加下面的配置export PATH="$HOME/.rbenv/bin:$PATH"eval "$(rbenv init -)"

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 谢谢指正，因为我的电脑已经有这些设置了，而且 CI 自动设置了也不报错。我们马上更新一下原文，再次感谢！

##### *奇：
> Ruby 环境配置打卡

##### *超：
> 老师您好我在安装rbenv的时候，一直遇到错误，连vpn都不行。日志如下== Downloading https://ghcr.io/v2/homebrew/core/m4/manifests/1.4.18-1Already downloaded: /Users/linchao/Library/Caches/Homebrew/downloads/aaf27e34b1345b8491869f9b867bcd141d4997760393586eb2b96a0a868ea049--m4-1.4.18-1.bottle_manifest.json== Downloading https://ghcr.io/v2/homebrew/core/m4/blobs/sha256:0df9083b268f76a3cda0c9f0d2ce84b51d21a8618d578740646fb615b00c7e7bcurl: (7) Couldn't connect to serverError: Failed to download resource "m4"Download failed: https://ghcr.io/v2/homebrew/core/m4/blobs/sha256:0df9083b268f76a3cda0c9f0d2ce84b51d21a8618d578740646fb615b00c7e7b求助老师。谢谢！

##### **力：
> 请问我用rbenv设置项目目录的ruby的版本是2.7.1 ， 为什么每次终端进入项目目录都提示ruby2.7.1 未安装，我电脑的是有2.7.1的版本。请问这种要怎么解决Required ruby-2.7.1 is not installed.To install do: 'rvm install "ruby-2.7.1"'

##### **鹏：
> 本地的 ruby --version 版本是：ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-darwin19]

##### **鹏：
> error: failed to download ruby-2.7.1.tar.bz2BUILD FAILED (macOS 11.1 using ruby-build 20210611)执行这个命令报错 rbenv install 2.7.1 我该怎么做

##### **力：
> 项目的.ruby-version 是2.7.1 ， Gemfile: Cocoapods版本指定是1.10.1 , 通过bundle install 命令安装，安装后通过bundle info cocoapods 命令打印详情信息，其中的Path是指向ruby2.4.0的版本，请问这是正常的么

##### **懿：
> 可以让所有开发人员使用 alias，在 terminal 中把 pod 改成 bundle exec pod，这样就避免了调用系统的 pod

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哇撒，你的方案很有创造性，赞！

##### **洋：
> Fetching gem metadata from https://rubygems.org/........google-api-client-0.38.0 requires ruby version ~ 2.4, which is incompatiblewith the current version, ruby 3.0.1p64Could not find CFPropertyList-3.0.2 in any of the sourcesRun `bundle install` to install missing gems.➜ iOS-linyongjian-main bundle installFetching gem metadata from https://rubygems.org/Fetching gem metadata from https://rubygems.org/........google-api-client-0.38.0 requires ruby version ~ 2.4, which is incompatiblewith the current version, ruby 3.0.1p64项目下载下来运行脚本报错M1 MacBook Air请老师指导下怎样在本台机子跑起来

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问是不是 rbevn 没有安装好呀？还是需要代理才能安装 Google API？

##### *明：
> 老师，执行gem install bundler命令进行，之后，再执行bundle init。这个是在根目录还是在项目目录下？如果要用git管理起来，是要在项目的目录下？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 都是在根目录下，可以参考 Moments App，它的 Gemfile 就在根目录 https://github.com/lagoueduCol/iOS-linyongjian

##### **5173：
> 说来惭愧，py都知道搞个env环境，但却忽视了iOS开发时环境的统一

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，很多问题可能是环境不统一引起的。有了统一的环境，可以节省大量环境搭建和调试问题的时间。

##### **5648：
> 可以用makefile 把常用的命令包装一下。比如执行make beta发布beta版本。这样可以避免本地环境或系统环境差异。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，makefile 也可以使用 xcodebuild 等命令来完成打包等操作，我们几年前也是这样做的。后来发现使用 fastlane 相对方便很多，我们的课程会讲述如何使用 fastlane 来封装常用的命令。原理上和你说的是一致的。

##### **7867：
> 老师，下面是我的使用流程 有不规范的地方吗？不同项目使用不同ruby 环境 和 不同版本的cocopods
1: 打开终端 cd ruby2.7.1 执行 rbenv install 2.7.1 和 rbenv local 2.7.1 会在当前文件夹下生成一个.ruby-version(默认是隐藏文件) 文件 里面显示 ruby 环境的版本信息2: 安装 Bundler">当前 终端 执行 gem install bundler 安装成功（第一次安装下次使用就不用安装了），执行bundler init 生成 Gemfile 文件 将你要使用的 cocopods 版本 写进去 例如 gem "cocoapods", "1.10.0”，执行 bundler install 安装 cocopods3: 在 ruby2.7.1 下 新建项目工程 使用 pod init ,pod install 就可以使用 版本 "1.10.0” 的 cocopods

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你的流程是对的，但记住在执行 pod 命令之前要执行 bundle exec 哦，否则它不会使用你的本地版本的 CocoaPods 啦。

##### *辰：
> 😊请问，CI是什么意思？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哦，CI 是 Continuous Integration，持续集成的意思，我们在第四部分会详细讲述这方面的内容。有了 CI，大量手工活可以让机器去做了。

##### **泽：
> 老师，如果项目开发环境改变了 比如.ruby-version的版本改变了， 是由负责人通知成员重新执行setup.sh，还是每次pull代码后执行一次呢。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 假如 .ruby-version的版本改变，当负责人把更新 push 到 Git 服务器后，通知大家 pull，然后执行一次 setup.sh 即可。以后不需要每次执行了。

##### **7867：
> gem install
ERROR: While executing gem ... (Gem::CommandLineError)
 Please specify at least one gem name (e.g. gem build GEMNAME)老师，一直报个错？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你是在哪里执行命令呢？是在项目的根目录吗？在该目录上应该一个 Gemfile 的，请再查

##### **龙：
> 不使用 rbenv ，那我们使用的 Ruby 来自Mac系统自带的。按上面说的配置cocopods版本的话，应该是可以使用不同的cocopods版本吧？只是我们一般都是全局的cocopods版本吧？问题还望老师解答。我觉得叫叉code挺好的，已经这么叫了很多年了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，如果不使用 rbenv 等 Ruby 环境工具就会使用系统自带的。为了保证整个团队都安装相同版本的 CocoaPods，有些极端的情况是把整个 CocoaPods 的源码都放到项目的 Git 里面。因为我们无法保证每个开发者的系统自带的 Ruby 的版本都是一样的，哪怕安装一样版本的 CocoaPods 也无法保证编译出来的 App 的行为是一样的。所以我们在文章中讲述了如何搭建一个统一的开发环境，这个环境是基于项目级别的并通过 Git 来管理。不同项目可以有自己独立的环境，当我们为项目升级 ruby，ruby gems 或者 Cocoapods 的版本时也不会相互影响。

##### *琦：
> 能否在gitee上做个镜像仓库，多谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我不使用 gitee 哦，如果你愿意帮忙，请帮我同步到 gitee 去。

##### *朋：
> bundle install 这个执行后 也是在全局环境装不通的版本吧。Gemfile 文件里面设置的版本同步的吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的， bundle install 执行后就拥有了项目级别的独立开发环境，我们项目里的 Gemfile 的版本只对该项目有效，和全局无关。这样做的好处是不会受到全局环境变动的影响，又能保证项目成员以及 CI 都使用统一的版本。

##### **东：
> macOS的版本统一是必须的吗，现在我们公司的系统版本并不是统一管理的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 根据我自己的经验，macOS 版本不一致问题不大。如果没办法保证 macOS 版本的统一，那么请保持 ruby 环境上所有版本的一致性，按照文章的方法可以搭建出来，而且只需要搭建一次。

##### **9987：
> 老师您好，通过一步步的敲代码发现以下问题，希望老师能够解答。1. 在setup.sh 中，第一步 `cat .ruby_version`，这里的路径是相对项目的路径，而不是home的路径，所以每次都会读取不到。是不是需要改为 `cat $HOME/.ruby_version` ？2. 还是setup.sh，在 bundle install执行后，直接执行了 bundle exec pod install。但是如果是新建的项目，按照github上的目录结构，此时应该是没有Podfile的。而且Gemfile所在目录没有.xcodeproj文件，也不能执行bundle exec pod init。进而也不会有.xcworkspace。所以在gem install以后，我该怎么生成Podfile和.xcworkspace文件呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. setup.sh 是在<Project_Dir> 下执行  ./scripts/setup.sh 。 2. 新项目需要按照文章的步骤搭建一次环境，其他开发人员才是执行 setup.sh。新项目是没有办法直接执行 setup.sh 的。

##### **斗：
> 牛牛

##### **泽：
> cd $(PROJECT_DIR)老师您好，这个是针对每个项目配置不同的环境是嘛？ 比如项目一 和 项目二使用不同的cocoapods版本 等，不知道我理解的对不对。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，我们希望每一个项目独立一个配置环境，相互不受影响。这样我们可以同时在不同的项目中工作，而这些项目的依赖库的版本都可以不一样，正如你说的项目一和项目二可以使用不同版本的 CocoaPods 或者 fastlane。

##### *伟：
> 新电脑如何安装 cocoapods 就直接跳过了？至少应该稍微提一下吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不好意思，我不是很理解你的问题呀，请问你说是执行 ./scripts/setup.sh 命令吗？这个命令是让团队的开发者可以一步搭建换，如果你想自己搭建环境，可以按照文章的步骤一步步来试试。

##### **千：
> 在安装 Bundler 时出错了，提示：➜ iOS-linyongjian git:(main) gem install bundlerERROR: While executing gem ... (Gem::FilePermissionError) You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.问题：没有权限修改系统的 Ruby/Gems 目录，怎么配置 gem 使用的是 rbenv 安装的ruby版本呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你安装了 rbenv 了吗？如果你没有安装 rbenv 就会使用系统的 ruby 环境，记得文章开头说过如果不配置统一的环境，会出现管理员权限的问题吗？感觉你碰到了这个问题了。如果你是在搭建 Moments App 的环境，请试一下执行  ./scripts/setup.sh 命令。

##### **0480：
> 搭建这个Ruby工具链是不是和系统的Ruby环境完全隔离的？就是说以前安装cocoapods环境不移除也不会和新搭建的工具链冲突？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，和系统的 Ruby 环境完全隔离了，以前的环境和新搭建的环境不会有任何冲突，因为新的环境是在一个 Sanbox 下面。每个项目都有自己独立的 Ruby 环境。还有提醒一下执行 CocoaPods 的命令时需要使用 bundle exec ，我一般在 Shell 里面设置一个alias be="bundle exec"，以后就可以使用  be pod install 啦。

##### **兵：
> 如果是不同的xCode版本会有什么问题么？我们最近也在做fastlane自动化。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 随着 Swift 语言稳定下来，一般 Xcode 的小版本更新不会引起什么问题，但还是推荐整个团队使用统一的版本，这样能保证编译出来 App 的运行效果能保持一致。

##### **豪：
> 林永坚Sliverlight For WP开发教程，这个是老师您的教程吗？从C#自学编程的时候，专门下载了这个教程来学Wp开发，不知道是不是老师您。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是我呀，想不到遇到老朋友啦，哈哈哈哈

##### **1810：
> 另外提个小小建议，Xcode 听读成叉Code感觉很别扭，能不能读成艾克斯Code啊。🤓

 ###### &nbsp;&nbsp;&nbsp; 官方客服回复：
> &nbsp;&nbsp;&nbsp; 好的，后续我们这边改改

##### **1810：
> 我们一直用系统自带的Ruby，xcodebuild 命令执行中会调用系统Ruby，用系统自带的Ruby也够用。

##### *瓜：
> RubyGems和Bundler这里有点绕，个人理解，RubyGems主要用来管理系统级别（全局的）gem依赖包，例如bundler这种；而Bundler则是用于管理不同项目下不同版本的gem依赖包，例如，项目A下使用的cocoapods版本是1.10.0， 项目B下使用的cocoapods 1.11.0 这种情况。

##### **彬：
> 系统有自带的ruby，但是版本比较低；我们现在使用的CocoaPods是我们自行下载并安装在系统/bin/local/下的，是从git上clong下载的；系统目录下不能同时安装多个版本的CocoaPods，一旦切换版本就会重新安装该版本。

##### **明：
> 大大小小也做过类似的事，总觉得不够系统。感谢林大大的分享。

##### **东：
> Failed to download resource "ruby-build"

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我 Google 了一下，可能和这个问题是一样的，需要使用 Homebrew 来重新安装 openssl 和 curl，详情请看 https://stackoverflow.com/a/45633751/749786 ，希望能帮到你哦。

##### jake：
> 用系统自带的ruby吧😀

