## NPM v5.x

- 宇宙最快(？貌似)，自带lock文件package-lock.json 来记录依赖树信息
- --save以后不用写了，会自动发保存到package.json，可以使用参数 --no-save取消
- node-gyp 在 Windows 提供对 node-gyp.cmd 的支持
- 包发布将同时生成 sha512 和 sha1 校验码，下载依赖时将使用强校验算法
- Git依赖支持优化，文件依赖优化，缓存优化

Note:
演示对比~
本次 npm5 新增了 package-lock.json 文件，在操作依赖时默认生成，用于记录和锁定依赖树的信息。使用过 yarn 的同学应该能感觉到，这次 npm5 的很多改动都有参考 yarn，这里估计也是在 yarn 的 lockfile 大受欢迎的背景下做出了这个修改（其实之前的 npm 版本并不是没有 lockfile）。

生成 package-lock.json 后，重复执行 npm install 时将会以其记录的版本来安装。这时如果手动修改 package.json 中的版本，重新安装也不会生效，只能手动执行 npm install 命令指定依赖版本来进行修改。而这一点 yarn 是可以做到的。猜想 yarn 在执行前是先对比了一遍 package.json 和 yarn.lock 中的版本，如果版本范围完全不符的话会重新安装并更新 lockfile。

npm5 新增的 package-lock.json 文件和之前通过 npm shrinkwrap 命令生成的 npm-shrinkwrap.json 文件的格式完全相同，文件内记录了版本，来源，树结构等所有依赖的 metadata。需要注意的是 npm shrinkwrap 并不是一个新功能特性，而是从 npm2 就开始有的功能。也就是说在 npm5 之前的版本也是可以通过 shrinkwrap 锁定依赖的。（在这一点上，其实 Facebook 也是早期在使用 npm shrinkwrap 等功能时无法满足需求才导致了现在 yarn 的出现。）

而最新的 npm5 在生成了 package-lock.json 之后，再运行 npm shrinkwrap 命令，会发现就是把 package-lock.json 重命名为 npm-shrinkwrap.json 而已。

因此 package-lock.json 表面上看只是把 npm-shrinkwrap.json 作为了默认创建，为何还要新建一个文件呢？官方对于此也给出了答复和解释：新增 package-lock.json 主要是为了使得 npm-shrinkwrap.json 可以向下兼容，保证旧版也可使用（比如已有 shrinkwrap 文件的项目，或是回滚旧版的场景）。另外 package-lock 的名称也比 shrinkwrap 相对更加直观。


![对比图](/images/nodejs8/npmvsyarn.png "NPM vs Yarn")

Note: 通过对比可以看出，yarn 的速度在大部分正常场景下还是略高一筹，不过相比之下 npm5 的差距已经很小。
yarn似乎强烈依赖于它的lock文件，而npm则主要依赖其内部的缓存。

单从数据来看最佳的搭配组合竟然为 yarn 和官方 registry，甚至更换 taobao registry 之后速度反而有所下降，目前还不知道 yarn 是否在自己的源上也有做了特殊优化。

注：单机测试样本不足，也可能有波动或误差，数据仅供参考，感兴趣的读者也可以用自己的项目和网络测一下。