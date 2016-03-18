---
layout: post
title: "优化push code到gerrit进行review"
description: ""
category: git
tags: [git]
---
新公司入职第三天，在熟悉了大致的代码结构时候，想尝试一次 code review 的过程，于是参考新人文档进行了相关操作。
在同事的帮助下，整体过程还算顺利，只是在提交review的时候遇到了几个坑，记录并处理如下：

####一.提交后无法在 code review 平台（gerrit）看到对应 review
在新人文档中说明了，提交review 的命令是：
{% highlight bash %}
$ git push origin HEAD:refs/for/master%r=xxx,r=yyy,cc=zzz,topic=xxxx # 注意这里的master是你希望最终提交代码的分支名
{% endhighlight %}
但是因为自己之前也用 gerrit，所以从以前的经验看，reviewer 和 topic 都是可有可无并且可随时在 web 平台添加的，于是，简单的使用了如下命令（ 同时gerrit 官方文档 支持短命令）

第一次尝试：
{% highlight bash %}
$ git push origin HEAD:refs/for/master
{% endhighlight %}

命令行提示成功

{% highlight bash %}
Counting objects: 55, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (11/11), 772 bytes | 0 bytes/s, done.
Total 11 (delta 5), reused 0 (delta 0)
remote: Resolving deltas: 100% (5/5)
remote: Counting objects: 110890, done
remote: Processing changes: closed: 2821, refs: 1, done   
To xxxx.xxxx.com:xxxxx.git
 * [new branch]      HEAD -> ref/for/master
{% endhighlight %}

但是在 cr 平台看不到任何提交，询问之后大家都是统一使用的完整命令，于是我又试了一下只增加 reviewer（没加 topic，因为在 cr 平台看到其他工程几乎都没有 topic）。

第二次尝试：

{% highlight bash %}
$ git push origin HEAD:refs/for/nmaster%r=xxxx, r=yyyy
{% endhighlight %}

依然成功，也提示了一行 **\* [new branch]** 的信息，同时 cr 平台还是没有提交信息。
这时候意识到问题可能是我的 change 没有提到 cr 平台，而是如 terminal 提示所说，在 remote 端建立了新的 remote branch，于是马上切到空的分支 git pull 一下，如下是 git branch -a 的部分结果：

{% highlight bash %}
remotes/origin/ref/for/master
remotes/origin/ref/for/master%r=xxxx,r=yyyy
remotes/origin/ref/for/release
remotes/origin/ref/for/release_aaa
{% endhighlight %}

果然，其中第二条恰好正是我提交命中显示的信息，而且猜测另外几个应该也是之前其他同事像我使用第一条命令一样生成的错误结果。
这次需要试试完整命令了。

第三次尝试：

{% highlight bash %}
$ git push origin HEAD:refs/for/news_master%r=xxxx,r=yyyy,cc=zzzz,topic=xxxxx
{% endhighlight %}

成功了，命令行显示信息是不同的：

{% highlight bash %}
Counting objects: 1, done.
Writing objects: 100% (1/1), 253 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
remote: Processing changes: new: 1, refs: 1, done   
remote: 
remote: New Changes:
remote:   https://gerrit.com/12345 Fix an error.
remote: 
To xxxx.xxxx.com:xxxxx.git
 * [new branch]      HEAD -> refs/for/master%r=xxxx,r=yyyy,cc=zzzz,topic=xxxx
{% endhighlight %}

注意这里依然写着 **[new branch]** ，但是前面有 **New Changes** 并列出 review 地址，说明一下后面的 new branch 是 gerrit 生成的（见 gerrit ），不是真正的 remote branch。
打开 cr 平台成功看到了 review 中的change，终于成功了。

问题：
1. 不知道是否是我们在 gerrit 平台上特意做的限定，所以必须增加了 reviewer 和 topic 之后才能提交 review ？
2. 其他 project 的 review 不知道为什么没有 topic，除非他们并不是按上述方式做的提交，或者项目配置不同。

####二. 尝试简化上述过程

如果每次提交代码都是要输入这么长的命令，缺点太多：

1. 效率太低
2. 有潜在操作 remote branch 的风险
3. reviewer 本身是随时可添加的，没必要每次提交时增加
4. 并不一定每个 change 都想让人 review，可能只是自己临时看，并不 merge
5. topic 目前来看，我们好像用不到

查了几个文档之后，在不改变当前 gerrit 对项目配置的情况，我做了如下改动：
在当前项目根目录的 .git/config 文件中增加如下内容：

{% highlight bash %}
[remote "review"]
    pushurl = ssh://xxx@gerrit.com:29418/xxxx    # 29418是 gerrit 的默认端口号，在平台上每个 change copy 的详情上可确认该端口号
    push = HEAD:refs/for/[master]                              # 最后中括号里是需要进行 review 之后提交的分支名，我这里写的 master
    receivepack = git receive-pack --reviewer xxx                # 最后是需要的 reviewer 名字，我这里把整行注释了，也建议大家注释，后面说明
{% endhighlight %}

之后再提交 review ，使用的命令如下：

{% highlight bash %}
$ git push review
{% endhighlight %}

直接成功，打开  gerrit.com 可看到新的提交，同时所有的 reviewer，topic 如果需要的话，可在 change 的右上方自由添加，不再截图。
上面的操作，对比之前的命令：git push orgin \*\*\*，这里的 origin 我们通过看 .git/config 会发现，它对应的是：

{% highlight bash %}
[remote "origin"]
    url = git.xxx.com:xxx.git
    fetch = +refs/heads/*:refs/remotes/origin/*
{% endhighlight %}

实际上它直接对应了我们 remote 端的代码库。而 **[remote "review"]** 中指定了 pushurl 是 gerrit.com，直接把我们的 change 抛给 gerrit 处理，完全不会影响到 remote 分支，之后 review 结束之后，在 cr 平台进行对应 cherry-pick、merge、rebase 等操作才会通过 gerrit 来操作 remote 代码，相对安全很多。
当然，我这里只是配置的 push 到 master，毕竟大家平时要在多分支开发，目前这个形式的方式是可以在 .git/config 里配置多个，如：

{% highlight bash %}
[remote "reviewD"]
    pushurl = ssh://xxxx@gerrit.com:29418/xxxx   
    push = HEAD:refs/for/[dev] 
{% endhighlight %}

想要 push 到 news_dev 分支，只需要使用：

{% highlight bash %}
$ git push reviewD
{% endhighlight %}

理论上应该是有可以不用每次手动增加分支配置的方式的，比如之前使用 repo 的时候会更自如很多，这个后续再研究吧，如果大家还有更恰当高效的方法，欢迎分享。

####三. 终极方案来了（两天后更新）
上面的方案二只是一个直观的简化当前 branch 提交操作的方式，今天在和同事聊天的时候，发现大家都希望的是不必每次都需要改 .git/config，特别是经常在多个 branch 切换的情况下。
这其实也是我考虑到的，因为刚入职，还在熟悉代码，所以只把眼前的情况给解决了，既然希望更彻底的解放劳动力，那还是继续完善方案吧，于是有了如下修改，直接看结果吧。

第一步，沿用前面的技术方案，但是完全不用上面的配置操作，一切重新来配置，第一步，使用下面的 **post-checkout**.hook 文件：

{% highlight bash %}
#!/bin/sh
#
# An example hook script to update current push setting for code reviewing.

# Current remote branch
current_branch=$(git name-rev --name-only HEAD)
current_remote=$(git config branch.$current_branch.merge)
#echo $current_remote

# Current remote repo
remote_repo=$(git config remote.origin.url)
# Current user
current_useremail=$(git config user.email)

# filter repo, branch and user using awk
repo_name=$(echo $remote_repo | awk -F ':' '{split($2,a,"."); print a[1]}')
#echo $repo_name

branch_name=$(echo $current_remote | awk -F '/' '{print $NF}')
#echo $branch_name

user=$(echo $current_useremail | awk -F '@' '{print $1}')
#echo $user

# Try to config current review push config
config_url="ssh://"$user"@gerrit.com:29418/"$repo_name
config_push="HEAD:refs/for/"$branch_name
echo "aurrent branch push review command is:"
echo "git push "$config_url $config_push
exec `git config remote.review.pushurl $config_url`
exec `git config remote.review.push $config_push`
{% endhighlight %}

相信大家在入职第一天配置环境的时候，为了 code review 都配置了 commit-msg 为了能增加 change-id，这里的 post-checkout 和 commit-msg 一样，需要放在你的代码库根目录的 .git/hooks/ 目录下，他和 commit-msg 一样是 git 的 hook 脚本，不需要做任何修改（除非大家的 review 平台不一样或其他特殊问题）。

第二步，同样的为它增加可执行权限:

{% highlight bash %}
$ chmod u+x post-checkout
{% endhighlight %}

第三步，（**只需要做一次**）对于当前分支需要先 checkout 到其他分支，再 checkout 回来才成生效。
（这里不一定是在 terminal 输入 checkout 命令，任何工具的切分支操作都可以）

第四步，同方案二里提到的，当需要提交 review 时，只需要执行：

{% highlight bash %}
$ git push review
{% endhighlight %}

说明：

1. 如果在 push 的时候出现需要输入密码，或者提示密码不对、无权限等问题，则需要检查自己的 ~/.ssh/config 配置的是否正确。
2. 终极方案中，每次在 terminal 中 checkout 后会提示在当前分支进行 push 时的完整命令，直接复制可用。但每次 checkout 都会提示，可以在 hook 里去掉。 
3. 终极方案中，如果不喜欢 “review” 这个名字，就需要修改 post-checkout 的 hook 文件，把 最后的两个 "review" 该为任意自己喜欢的单词。

