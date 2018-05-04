# How to compile .sol in Web3.py without 'py-solc'

These days I am trying to build a DAPP with Ethereum Web3 API, but I am not familiar with JavaScript. Thus, I choose Web3.py as my tools. However, I can not even run the example of the Web3.py's tutorial. The problem is that py-solc can hardly find my solidity compiler's path.

I have tried servral ways to solve it but all failed, so I write a small script to compile the solidity and replace the compile source function in the example.

The script uses the solcjs, please install it firstly with npm:

```shell
npm install -g solc
```

The return of this script is a dictionary with 'abi' key and 'bin' key which are used in the example. The whole example is shown in my github project https://github.com/liu-hanwen/ETH-Based-AirTicket/blob/master/web3py_test.py.

```Python
def compile_source(src, log=False):
    if log:
        print('Start to compile source...')
    filename = str(hash(src))+'.sol'
    files_before = subprocess.check_output('ls').decode('utf8').split('\n')
    with open(filename, 'w') as f:
        f.write(src)
    subprocess.check_call("solcjs --abi --bin %s \n exit 0" % filename, shell=True)
    ret = {}

    files_after = subprocess.check_output('ls').decode('utf8').split('\n')
    files_same = set(files_before) & set(files_after)

    targets = [file for file in files_after if file not in files_same]
    for target in targets:
        if '.abi' == target[-4:]:
            with open(target, 'r') as f:
                ret['abi'] = json.load(f)
        elif '.bin' == target[-4:]:
            with open(target, 'r') as f:
                ret['bin'] = f.read()
        else:
            continue

        subprocess.check_call('rm %s' % target, shell=True)

    subprocess.check_call('rm %s' % filename, shell=True)
    if log:
        print('Compiled!\nThe return is:\n' , ret)
    return ret
```

---

# 配不好py-solc怎么编译.sol并使用Web3.py

最近因为课程需要想写一个基于以太坊智能合约的DAPP，因为不会用js所以就想用web3.py，可是就连tutorial里面最基础的例子我都不能运行，py-solc库老是说找不到solc。我上网找了好久也没找到解决方案，最后干脆就用solc的js版本先编译然后再传到python里面好了。下面是我写的一个小脚本，这个脚本用solcjs编译了.sol然后返回了一个和例子里面一样的字典，相当于替代了py-solc里面的compile-source函数。

这个小脚本需要先安装solcjs：

```shell
npm install -g solc
```

脚本和tutorial例子整合的示例在我的项目里：https://github.com/liu-hanwen/ETH-Based-AirTicket/blob/master/web3py_test.py ，下面是替代py-solc的compile_source的脚本函数。

```Python
def compile_source(src, log=False):
    if log:
        print('Start to compile source...')
    filename = str(hash(src))+'.sol'
    files_before = subprocess.check_output('ls').decode('utf8').split('\n')
    with open(filename, 'w') as f:
        f.write(src)
    subprocess.check_call("solcjs --abi --bin %s \n exit 0" % filename, shell=True)
    ret = {}

    files_after = subprocess.check_output('ls').decode('utf8').split('\n')
    files_same = set(files_before) & set(files_after)

    targets = [file for file in files_after if file not in files_same]
    for target in targets:
        if '.abi' == target[-4:]:
            with open(target, 'r') as f:
                ret['abi'] = json.load(f)
        elif '.bin' == target[-4:]:
            with open(target, 'r') as f:
                ret['bin'] = f.read()
        else:
            continue

        subprocess.check_call('rm %s' % target, shell=True)

    subprocess.check_call('rm %s' % filename, shell=True)
    if log:
        print('Compiled!\nThe return is:\n' , ret)
    return ret
```

