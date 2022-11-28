## Tutorial Testnet Mina Protocol 

- [Discord](https://discord.gg/6R44epBHsh) 
- [docs Mina ](https://docs.minaprotocol.com/zkapps/tutorials/zkapp-ui-with-react) 
- [Wallet](https://www.aurowallet.com/) | [Faucet](https://faucet.minaprotocol.com/) 
- [minascan](https://minascan.io/berkeley/validators/leaderboard) | [mina explorer](https://berkeley.minaexplorer.com/)
- [Rewards Detail](https://minaprotocol.com/blog/zkspark-cohort0?_hsenc=p2ANqtz-8smwqFrO-bZbm3_8-KWLkOJEV5_-yyWKkPzNswcOViTtGGAsJ2Ixg_W6Efo0kaIah9zr_wPl3trIgYeeJwCA40SGbKOQ&_hsmi=234896730)

## Spesifikasi Minimum
| Komponen  | Requirements minimal|
|-----------|---------------------|
|Sistem Operasi|Ubuntu 16.04 - 20.04 |
|CPU           | 4 Cores             |
|RAM           | 64GB                |
|Penyimpanan   | 1TB (SSD atau NVME) |
|Koneksi       |10Mbps - 100Mbps     |

## Install dan Jalankan
- Install wallet dan Faucet terlebih dahulu
### Open port
```
ufw allow 22 && ufw allow 3000
ufw enable
```
## Instal Node Js 16 & NPM & Git
```
sudo apt update
sudo apt upgrade
```
```
sudo apt install git
sudo apt install -y curl
```
```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```
```
sudo apt install -y nodejs
```

## Install Zkapp CLI

```
git clone https://github.com/o1-labs/zkapp-cli
```
```
npm instal -g zkapp-cli@0.5.3
```

Check Version 
```
zk --version
```

## Buat Project

```
zk project 04-zkapp-browser-ui --ui next
```

Konfirmasi ***Yes*** 3x dan `Enter` Biarkan Hingga Proses Instalisasi Selesai 

## Buat Repository 

Buat repo di Github anda dengan nama `04-zkapp-browser-ui` dan simpan

[![Screenshot-2022-11-28-151725.png](https://i.postimg.cc/brM59nfP/Screenshot-2022-11-28-151725.png)](https://postimg.cc/gLV4Yn0t)

## Kembali ke terminal 

### Remote Github

```
cd 04-zkapp-browser-ui
```
```
git remote add origin Ganti-URL-Repository-anda
```
```
git push -u origin main
```
`Ganti-URL-Repository-anda` contoh 👇

[![Screenshot-2022-11-28-153215.png](https://i.postimg.cc/XJnXnZhn/Screenshot-2022-11-28-153215.png)](https://postimg.cc/Z9DJcq4Q)

Setelah memasukan command terakhir Masukan Username & Password Github anda

jika login gagal/eror silahkan buat ***Token Acces*** terlebih dahulu  Pengganti Password

- [Tutorial membuat access token Github](https://art-sy5team.gitbook.io/access-token-or-github/)

## Run contracts

```
cd
cd 04-zkapp-browser-ui/contracts/
npm run build
```

## Membuat 2 File Baru 

```
cd
cd 04-zkapp-browser-ui/ui/pages
```

### Buat 2 File Baru Berisi :

##### Folder Pertama : 
```
nano zkappWorker.ts
```
Copy dan Paste Isian Script Ini di Terminal VPS  Kalian :

```
import {
    Mina,
    isReady,
    PublicKey,
    PrivateKey,
    Field,
    fetchAccount,
} from 'snarkyjs'

type Transaction = Awaited<ReturnType<typeof Mina.transaction>>;

// ---------------------------------------------------------------------------------------

import type { Add } from '../../contracts/src/Add';

const state = {
    Add: null as null | typeof Add,
    zkapp: null as null | Add,
    transaction: null as null | Transaction,
}

// ---------------------------------------------------------------------------------------

const functions = {
    loadSnarkyJS: async (args: {}) => {
        await isReady;
    },
    setActiveInstanceToBerkeley: async (args: {}) => {
        const Berkeley = Mina.BerkeleyQANet(
            "https://proxy.berkeley.minaexplorer.com/graphql"
        );
        Mina.setActiveInstance(Berkeley);
    },
    loadContract: async (args: {}) => {
        const { Add } = await import('../../contracts/build/src/Add.js');
        state.Add = Add;
    },
    compileContract: async (args: {}) => {
        await state.Add!.compile();
    },
    fetchAccount: async (args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        return await fetchAccount({ publicKey });
    },
    initZkappInstance: async (args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        state.zkapp = new state.Add!(publicKey);
    },
    getNum: async (args: {}) => {
        const currentNum = await state.zkapp!.num.get();
        return JSON.stringify(currentNum.toJSON());
    },
    createUpdateTransaction: async (args: {}) => {
        const transaction = await Mina.transaction(() => {
            state.zkapp!.update();
        }
        );
        state.transaction = transaction;
    },
    proveUpdateTransaction: async (args: {}) => {
        await state.transaction!.prove();
    },
    getTransactionJSON: async (args: {}) => {
        return state.transaction!.toJSON();
    },
};

// ---------------------------------------------------------------------------------------

export type WorkerFunctions = keyof typeof functions;

export type ZkappWorkerRequest = {
    id: number,
    fn: WorkerFunctions,
    args: any
}

export type ZkappWorkerReponse = {
    id: number,
    data: any
}
if (process.browser) {
    addEventListener('message', async (event: MessageEvent<ZkappWorkerRequest>) => {
        const returnData = await functions[event.data.fn](event.data.args);

        const message: ZkappWorkerReponse = {
            id: event.data.id,
            data: returnData,
        }
        postMessage(message)
    });
}
```

Simpan `CTRL` `X` `Y` dan `Enter`

##### Folder Kedua :

```
nano zkappWorkerClient.ts
```

Copy dan Paste Isian Script Ini di Terminal VPS Kalian :

```
import {
    fetchAccount,
    PublicKey,
    PrivateKey,
    Field,
} from 'snarkyjs'

import type { ZkappWorkerRequest, ZkappWorkerReponse, WorkerFunctions } from './zkappWorker';

export default class ZkappWorkerClient {

    // ---------------------------------------------------------------------------------------

    loadSnarkyJS() {
        return this._call('loadSnarkyJS', {});
    }

    setActiveInstanceToBerkeley() {
        return this._call('setActiveInstanceToBerkeley', {});
    }

    loadContract() {
        return this._call('loadContract', {});
    }

    compileContract() {
        return this._call('compileContract', {});
    }

    fetchAccount({ publicKey }: { publicKey: PublicKey }): ReturnType<typeof fetchAccount> {
        const result = this._call('fetchAccount', { publicKey58: publicKey.toBase58() });
        return (result as ReturnType<typeof fetchAccount>);
    }

    initZkappInstance(publicKey: PublicKey) {
        return this._call('initZkappInstance', { publicKey58: publicKey.toBase58() });
    }

    async getNum(): Promise<Field> {
        const result = await this._call('getNum', {});
        return Field.fromJSON(JSON.parse(result as string));
    }

    createUpdateTransaction() {
        return this._call('createUpdateTransaction', {});
    }

    proveUpdateTransaction() {
        return this._call('proveUpdateTransaction', {});
    }

    async getTransactionJSON() {
        const result = await this._call('getTransactionJSON', {});
        return result;
    }

    // ---------------------------------------------------------------------------------------

    worker: Worker;

    promises: { [id: number]: { resolve: (res: any) => void, reject: (err: any) => void } };

    nextId: number;

    constructor() {
        this.worker = new Worker(new URL('./zkappWorker.ts', import.meta.url))
        this.promises = {};
        this.nextId = 0;

        this.worker.onmessage = (event: MessageEvent<ZkappWorkerReponse>) => {
            this.promises[event.data.id].resolve(event.data.data);
            delete this.promises[event.data.id];
        };
    }

    _call(fn: WorkerFunctions, args: any) {
        return new Promise((resolve, reject) => {
            this.promises[this.nextId] = { resolve, reject }

            const message: ZkappWorkerRequest = {
                id: this.nextId,
                fn,
                args,
            };

            this.worker.postMessage(message);

            this.nextId++;
        });
    }
}
```

Simpan `CTRL` `X` `Y` dan `Enter`

## 8 . Tambahkan Css

```
cd
cd 04-zkapp-browser-ui/ui/styles
```

Kemudian

```
nano globals.css
```

Hapus Semu Isi Yang Ada di Sana dan Ganti Dengan Script di Bawah Ini :

```
html,
body {
  padding: 0;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen,
    Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
}

a {
  color: inherit;
  text-decoration: none;
}

* {
  box-sizing: border-box;
}

@media (prefers-color-scheme: dark) {
  html {
    color-scheme: dark;
  }
  body {
    color: white;
    background: black;
  }
}
```

Simpan `CTRL` `X` `Y` dan `Enter`

## 10 .  Running Web Buka 2 Terminal Baru (1 Perintah Masing Masing 1 Tab)

##### TAB 1
```
cd
cd 04-zkapp-browser-ui/ui/
npm run dev
```

##### TAB 2

```
cd
cd 04-zkapp-browser-ui/ui/
npm run ts-watch
```

1. Lalu Buka Browser Chrome http://ip-vps-kalian:3000
2. Pastikan Snarkjs Terbuka (Walaupun Error Abaikan Saja)
3. Lalu 2 Tab Tersebut Jika Sudah Jalan, Bisa Kalian Close Langsung Atau Kalian Bisa Menjalankan Tab Baru

## 11 . Implementing the react app

```
cd
cd 04-zkapp-browser-ui/ui/pages
nano _app.page.tsx
```

Hapus Command Yang Ada di VPS kalian Ganti Dengan Script di Bawah ini :

```
// import '../styles/globals.css'
// import type { AppProps } from 'next/app'

// import './reactCOIServiceWorker';

// export default function App({ Component, pageProps }: AppProps) {
//   return <Component {...pageProps} />
// }


import '../styles/globals.css'
import { useEffect, useState } from "react";
import './reactCOIServiceWorker';

import ZkappWorkerClient from './zkappWorkerClient';

import {
  PublicKey,
  PrivateKey,
  Field,
} from 'snarkyjs'

let transactionFee = 0.1;

export default function App() {

  let [state, setState] = useState({
    zkappWorkerClient: null as null | ZkappWorkerClient,
    hasWallet: null as null | boolean,
    hasBeenSetup: false,
    accountExists: false,
    currentNum: null as null | Field,
    publicKey: null as null | PublicKey,
    zkappPublicKey: null as null | PublicKey,
    creatingTransaction: false,
  });

  // -------------------------------------------------------
  // Do Setup

  useEffect(() => {
    (async () => {
      if (!state.hasBeenSetup) {
        const zkappWorkerClient = new ZkappWorkerClient();

        console.log('Loading SnarkyJS...');
        await zkappWorkerClient.loadSnarkyJS();
        console.log('done');

        await zkappWorkerClient.setActiveInstanceToBerkeley();

        const mina = (window as any).mina;

        if (mina == null) {
          setState({ ...state, hasWallet: false });
          return;
        }

        const publicKeyBase58: string = (await mina.requestAccounts())[0];
        const publicKey = PublicKey.fromBase58(publicKeyBase58);

        console.log('using key', publicKey.toBase58());

        console.log('checking if account exists...');
        const res = await zkappWorkerClient.fetchAccount({ publicKey: publicKey! });
        const accountExists = res.error == null;

        await zkappWorkerClient.loadContract();

        console.log('compiling zkApp');
        await zkappWorkerClient.compileContract();
        console.log('zkApp compiled');

        const zkappPublicKey = PublicKey.fromBase58('B62qrDe16LotjQhPRMwG12xZ8Yf5ES8ehNzZ25toJV28tE9FmeGq23A');

        await zkappWorkerClient.initZkappInstance(zkappPublicKey);

        console.log('getting zkApp state...');
        await zkappWorkerClient.fetchAccount({ publicKey: zkappPublicKey })
        const currentNum = await zkappWorkerClient.getNum();
        console.log('current state:', currentNum.toString());

        setState({
          ...state,
          zkappWorkerClient,
          hasWallet: true,
          hasBeenSetup: true,
          publicKey,
          zkappPublicKey,
          accountExists,
          currentNum
        });
      }
    })();
  }, []);

  // -------------------------------------------------------
  // Wait for account to exist, if it didn't

  useEffect(() => {
    (async () => {
      if (state.hasBeenSetup && !state.accountExists) {
        for (; ;) {
          console.log('checking if account exists...');
          const res = await state.zkappWorkerClient!.fetchAccount({ publicKey: state.publicKey! })
          const accountExists = res.error == null;
          if (accountExists) {
            break;
          }
          await new Promise((resolve) => setTimeout(resolve, 5000));
        }
        setState({ ...state, accountExists: true });
      }
    })();
  }, [state.hasBeenSetup]);

  // -------------------------------------------------------
  // Send a transaction

  const onSendTransaction = async () => {
    setState({ ...state, creatingTransaction: true });
    console.log('sending a transaction...');

    await state.zkappWorkerClient!.fetchAccount({ publicKey: state.publicKey! });

    await state.zkappWorkerClient!.createUpdateTransaction();

    console.log('creating proof...');
    await state.zkappWorkerClient!.proveUpdateTransaction();

    console.log('getting Transaction JSON...');
    const transactionJSON = await state.zkappWorkerClient!.getTransactionJSON()

    console.log('requesting send transaction...');
    const { hash } = await (window as any).mina.sendTransaction({
      transaction: transactionJSON,
      feePayer: {
        fee: transactionFee,
        memo: '',
      },
    });

    console.log(
      'See transaction at https://berkeley.minaexplorer.com/transaction/' + hash
    );

    setState({ ...state, creatingTransaction: false });
  }

  // -------------------------------------------------------
  // Refresh the current state

  const onRefreshCurrentNum = async () => {
    console.log('getting zkApp state...');
    await state.zkappWorkerClient!.fetchAccount({ publicKey: state.zkappPublicKey! })
    const currentNum = await state.zkappWorkerClient!.getNum();
    console.log('current state:', currentNum.toString());

    setState({ ...state, currentNum });
  }

  // -------------------------------------------------------
  // Create UI elements

  let hasWallet;
  if (state.hasWallet != null && !state.hasWallet) {
    const auroLink = 'https://www.aurowallet.com/';
    const auroLinkElem = <a href={auroLink} target="_blank" rel="noreferrer"> [Link] </a>
    hasWallet = <div> Could not find a wallet. Install Auro wallet here: {auroLinkElem}</div>
  }

  let setupText = state.hasBeenSetup ? 'SnarkyJS Ready' : 'Setting up SnarkyJS...';
  let setup = <div> {setupText} {hasWallet}</div>

  let accountDoesNotExist;
  if (state.hasBeenSetup && !state.accountExists) {
    const faucetLink = "https://faucet.minaprotocol.com/?address=" + state.publicKey!.toBase58();
    accountDoesNotExist = <div>
      Account does not exist. Please visit the faucet to fund this account
      <a href={faucetLink} target="_blank" rel="noreferrer"> [Link] </a>
    </div>
  }

  let mainContent;
  if (state.hasBeenSetup && state.accountExists) {
    mainContent = <div>
      <button onClick={onSendTransaction} disabled={state.creatingTransaction}> Send Transaction </button>
      <div> Current Number in zkApp: {state.currentNum!.toString()} </div>
      <button onClick={onRefreshCurrentNum}> Get Latest State </button>
    </div>
  }

  return <div>
    {setup}
    {accountDoesNotExist}
    {mainContent}
  </div>
}
```

## 12 . Deploy UI ke Repository Kalian

```
cd 04-zkapp-browser-ui/ui/
npm run deploy
```

Jika ada Output Error Seperti Ini Abaikan Dan Tunggu Hingga Proses Selesai :

`./pages/_app.page.tsx
95:6  Warning: React Hook useEffect has a missing dependency: 'state'. Either include it or remove the dependency array. You can also do a functional update 'setState(s => ...)' if you only need 'state' in the 'setState' call.  react-hooks/exhaustive-deps
115:6  Warning: React Hook useEffect has a missing dependency: 'state'. Either include it or remove the dependency array. You can also do a functional update 'setState(s => ...)' if you only need 'state' in the 'setState' call.  react-hooks/exhaustive-deps`

1. Nanti Ketika Selesai Anda Akan di Suruh Memasukan Username Github Kalian
2. Masukan Password Github Anda, Ingat di Awal Kita Menggunakan Token Acces. Gunakan Itu Lagi Untuk Password Github (Atau Buat Token Acces Baru Lagi)

## Untuk Memastikan Apakah Sudah Jalan Dengan Baik

1. Buka dan Edit : `https://<Your-Username>.github.io/04-zkapp-browser-ui/index.html`
2. Your-Username = Ganti Dengan Nama Github Kalian
3. Jika Sudah, Paste ke Google Chrome
4. Jika Berjalan Dengan normal Akan Terbuka dan Meminta Aprrove Transaksi dari Wallet Mina Kalian

## Send Beberapa Transaksi 

1. Buka `https://<Your-Username>.github.io/04-zkapp-browser-ui/index.html` Kalian
2. Connect Wallet Mina nya dan Approve
3. Refresh Web Tunggu Sampe Muncul Tombol `Send Transaksi` 
4. Tunggu Hingga Muncul Pop Up Approve Transaksi > ***Isi Fee 1***
5. Membutuhkan beberapa waktu untuk muncul, setelah transaction selesai check explorer
6. lakukan berulang hingga transaction berhasil ***hijau***
7. isi from Register : https://fisz9c4vvzj.typeform.com/zkSparkTutorial

[![Txhash.png](https://i.postimg.cc/4d82XSPw/Txhash.png)](https://postimg.cc/tZxN5DLV)

## 16 . Uninstal Semua (Jika Ingin Menghapus)

```
rm -rf 04-zkapp-browser-ui
rm -rf zkapp-cli
rm -rf .npm
npm uninstall -g zkapp-cli
sudo apt-get remove nodejs
```

### Art-Team INFO
noted: ***art team*** here does not have any specific goals or intentions, they only collect data and share it with everyone.

untuk INFO Testnet lainya Silahkan join Discord 👇

[![twitter](https://img.shields.io/badge/twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://twitter.com/ArtSy5team)
[![Discord](https://img.shields.io/badge/discord-7289d9?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/EAKEdZU6c8)
[![Github](https://img.shields.io/badge/GitHub-171515?style=for-the-badge&logo=GitHub&logoColor=white)](https://github.com/Art-Sy5team)
[![trakteer](https://img.shields.io/badge/trakteer.id-e31e1e?style=for-the-badge&logo=ko-fi&logoColor=white)](https://trakteer.id/Art-Sy5team/tip)
