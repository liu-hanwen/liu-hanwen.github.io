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

