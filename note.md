# git指令

- [git指令](#git指令)
  - [git merge](#git-merge)
    - [merge的讲解](#merge的讲解)
      - [1.指令基础](#1指令基础)
      - [2.fast-forward](#2fast-forward)
      - [3.冲突与处理](#3冲突与处理)
      - [4.预处理](#4预处理)
      - [5.diff-algorithm](#5diff-algorithm)
      - [6.重命名的merge](#6重命名的merge)
      - [7.合并策略](#7合并策略)
    - [merge的练习](#merge的练习)
      - [无修改删除,只有增添文件情况下的merge](#无修改删除只有增添文件情况下的merge)

## git merge

### merge的讲解

#### 1.指令基础

- merge指令是要求git将两个commit的内容进行合并,
  - 这两个commit一个记为ours,一个记为theirs
    - **ours**默认是当前HEAD指向的分支的最后一个commit
    - **theirs**是通过参数指定的非HEAD分支中的某一个commit
      - **merge hashCode**  # theirs 是hashCode标记的某一个commit
      - **merge BranchName** # theirs 默认是BranchName分支最新的commit

#### 2.fast-forward

所谓的fast-forward是指两个commit在分支树上具有**父子关系**,而非兄弟关系.\
对于merge指令来说,当合并的commit是当前HEAD的子孙时,称为fast-forward.\
这样的merge操作不会产生commit,而是仅仅移动一下HEAD指针即可.

- merge的与fast-forward相关的可选参数:
- **--ff**
    When the merge resolves as a fast-forward, only update the branch\
    pointer, without creating a merge commit. This is the default\
    behavior.

- **--no-ff**
     Create a merge commit even when the merge resolves as a\
     fast-forward. This is the default behaviour when merging an\
     annotated (and possibly signed) tag.

#### 3.冲突与处理

- git的三路合并算法:
git在尝试合并两个commit的时候,会执行三路合并算法.三路合并算法在执行的时候,\
会选择两个commit的最年轻的公共祖先commit作为base,分别比对两个commit的变化:
  - 两个commit相对于base的变化是一样的----不存在冲突
  - 两个commit相对于base一个变化了,一个无变化----不存在冲突
  - 两个commit相对于base的变化是不一样的----存在冲突,需要人为解决

- git的递归三路合并:
因为合并指令的存在,两个commit可能不仅仅有唯一一个最年轻的公共祖先. \
称这种公共祖先不确定的情况为**criss-cross merge ambiguities**,算法需要额外确定公共祖先\
此时将可能的祖先commits进行merge得到最佳祖先commit(虚拟的commit,并不真实存在).\
称这种需要沿着版本树进行多次merge操作的合并算法为递归三路合并:

  - 案例:
    - merge s2 to s1 ---> o1
    - merge s1 to s2 ---> o2
    - o1 ---> A
    - o2 ---> o3 ---> B
  - 如上所示,合并A和B的时候s1和s2都是可能的最佳公共祖先.
  - 递归三路合并会将s1和s2先进行合并,虚拟出A和B的最佳虚拟祖先.

- 参数
  - **ours**

    This option forces conflicting hunks to be auto-resolved cleanly by favoring our version.\
    Changes from the other tree that do not conflict with our side are reflected in the merge\
    result. For a binary file, the entire contents are taken from our side.

    This should not be confused with the ours merge strategy, which does not even look at what\
    the other tree contains at all. It discards everything the other tree did, declaring our\
    history contains all that happened in it.

  - **theirs**

    This is the opposite of ours; note that, unlike ours, there is no theirs merge strategy to\
    confuse this merge option with.

#### 4.预处理

在merge之前,一般要保证working tree clean状态,也就是说不存在未staged & commited的修改.\
如果保证untracked的文件和不会造成冲突也可以不进行stage-commit;然而merge指令不允许存在\
已经staged但是尚未commit的文件.

- 修改未staged报错:\
error: Your local changes to the following files would be overwritten by merge:file1.txt\
Please commit your changes or stash them before you merge.
Aborting
- 修改未commited报错:\
error: Your local changes to the following files would be overwritten by merge:file1.txt\

#### 5.diff-algorithm

merge可以指定判断差别的签名算法,不同的算法效率和准确度差别较大:

- 可选算法:
  - patience   (最准确的算法)
  - minimal
  - histogram
  - myers  (默认的算法)

#### 6.重命名的merge

git的diff-algorithm可以识别文件重命名情况,因此无需担心因为重命名而导致的冲突.\
在git内部,每一个文件都是以不记名blob保存的,git额外保存一个文件到名称的映射关系.\
配置参数find-renames[=\<n\>]可以人为界定重命名阈值.

#### 7.合并策略

- octopus

    This resolves cases with more than two heads, but refuses to do a complex merge that\
    needs manual resolution. It is primarily meant to be used for bundling topic branch heads\
    together. This is the default merge strategy when pulling or merging more than one branch.\
- ours

    This resolves any number of heads, but the resulting tree of the merge is always that of the\
    current branch head, effectively ignoring all changes from all other branches. It is meant to\
    be used to supersede old development history of side branches. Note that this is different\
    from the -Xours option to the recursive merge strategy.
- subtree

    This is a modified recursive strategy. When merging trees A and B, if B corresponds to a subtree\
    of A, B is first adjusted to match the tree structure of A, instead of reading the trees at the same\
    level. This adjustment is also done to the common ancestor tree.
  - subtree[=\<path\>]\
        This option is a more advanced form of subtree strategy, where the strategy makes a guess on\
        how two trees must be shifted to match with each other when merging. Instead, the specified \
        path is prefixed (or stripped from the beginning) to make the shape of two trees to match.

### merge的练习

#### 无修改删除,只有增添文件情况下的merge

- 1.首先创建git 项目树为:
  - main:
    - add file1.txt   **# commit hashcode=h1**
    - add file2.txt   **# commit hashcode=h2**
- 2.创建新的分支,继承自main:
  - develop:
    - add file1.txt   **# commit hashcode=h1**
    - add file2.txt   **# commit hashcode=h2**
- 3.在新的分支中添加新的文件(原有文件不修改):
  - develop:
    - add file1.txt   **# commit hashcode=h1**
    - add file2.txt   **# commit hashcode=h2**
    - add file3.txt   **# commit hashcode=h3**
- 4.切换到main分支,并增添新的文件(原有文件不修改):
  - main:
    - add file1.txt       **# commit hashcode=h1**
    - add file2.txt       **# commit hashcode=h2**
    - add mainfile3.txt   **# commit hashcode=h4**
- 5.切换到develop分支,并增添新的文件(原有文件不修改):
  - develop:
    - add file1.txt   **# commit hashcode=h1**
    - add file2.txt   **# commit hashcode=h2**
    - add file3.txt   **# commit hashcode=h3**
    - add file4.txt   **# commit hashcode=h5**
- 6.切换到main分支,并且将develop分支的第三个commit(hashcode=h3)合并到main分支
  - 执行命令 git merge h3
  - 在main分支产生了新的commit,在其中将develop中添加的file3.txt加入到main分支之内
  - main:
    - add file1.txt       **# commit hashcode=h1**
    - add file2.txt       **# commit hashcode=h2**
    - add mainfile3.txt   **# commit hashcode=h4**
    - merge file3.txt from commit h3  **# commit hashcode=h6**
