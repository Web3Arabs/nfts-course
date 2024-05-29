# ربط مشروع NFT بالواجهة الامامية

بعد ان انتهينا من إنشاء مشروع NFT يمكننا تشغيله على موقعنا من اجل ان يتمكن المستخدم من التفاعل معه.

في هذا الدرس سنقوم بإستخدام إطار العمل Next.js و TailwindCSS من اجل بناء واجهة الموقع.

قم بتشغيل هذا الامر في المجلد **my-app** الذي قمنا بإنشائه سابقاً من اجل إنشاء مشروع nextjs

```bash
npx create-next-app@latest .
```

<img src="https://www.web3arabs.com/courses/nfts/w3arabsnft/create-nextjs.png"/>

سنحتاج الان الى تثبيت ethers.js والتي ستساعدنا في التعامل مع العقد الذكي وإرسال المعاملات. قم بكتابة هذا على terminal

```bash
npm install ethers@5
```

ستذهب الى مجلد app وستقوم بفتح الملف **globals.css** وستبقي هذه الاوامر في الملف

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

<img src="https://www.web3arabs.com/courses/nfts/w3arabsnft/globals-css.png"/>

الان اذهب الى الملف **page.js** في المجلد **app** وقم بلصق هذا الكود ومتابعة الشرح من التعليقات المتواجدة اعلى كل سطر.


```jsx
"use client"
import { useState, useEffect } from "react"
import { ethers } from "ethers"
import abi from "../../contract-tutorial/out/W3ArabsNFT.sol/W3ArabsNFT.json"

export default function Home() {
  // Contract Address & ABI
  const contractAddress = "add_your_smart_contract_address_here"
  const contractABI = abi.abi

  // ستقوم بتخزين عنوان المحفظة التي ستتصل بالتطبيق من اجل مراقبة ما اذا كانت المحفظة متصلة بالتطبيق او نتمكن من استدعاء عنوان المحفظة المتصلة
  const [currentAccount, setCurrentAccount] = useState("")
  // المقبوضة NFTs تخزين اجمالي عدد
  const [tokenIds, setTokenIds] = useState("0")
  // مالك العقد الذكي address تخزين
  const [owner, setOwner] = useState("")
  // التي يمتلكها الموقع NFTs تخزين اجمالي عدد
  const [balance, setBalance] = useState("0")

  // تعمل هذه الوظيفة على مراقبة اتصال المحفظة بالتطبيق بشكل مستمر
  const isConnectWallet = async () => {
    try {
      // يتم استخدام هذا للوصول إلى كائن اثيريوم والتي من تعد من الكائنات العامة
      const { ethereum } = window

      // يقوم هذا بإرجاع مجموعة من عناوين شبكة اثيريوم المرتبطة بحساب المستخدم ويمكن استخدام هذا للوصول إلى حسابات المستخدم اثيريوم والتفاعل مع شبكة اثيريوم
      const accounts = await ethereum.request({method: "eth_requestAccounts"})
      console.log("accounts: ", accounts)
      setCurrentAccount(accounts[0])

      /**
        يتحقق هذا الرمز من وجود أي حسابات متاحة في المحفظة الخاصة باللمستخدم.
        إذا كان هناك، فإنه يقوم بتعيين الحساب الأول إلى متغير يسمى ويطبع رسالة في وحدة التحكم.
        إذا لم تكن هناك حسابات متاح، فإنها تطبع رسالة في وحدة التحكم.
      */
      if (accounts.length > 0) {
        const account = accounts[0]
        console.log("wallet is connected! ", account)
      } else {
        console.log("make sure MetaMask is connected")
      }
    } catch (error) {
      console.error(error)
    }
  }

  // تعمل هذه الوظيفة على اتصال المحفظة بالتطبيق
  const connectWallet = async () => {
    try {
      // يتم استخدام هذا للوصول إلى كائن اثيريوم والتي من تعد من الكائنات العامة
      const { ethereum } = window

      // يتحقق هذا لمعرفة ما إذا كان موفر شبكة اثيريوم متوفراً. إذا لم يكن متوفراً, فسيخرج رسالة
      if (!ethereum) {
        console.log("please install MetaMask")
      }

      // يقوم هذا بإرجاع مجموعة من عناوين شبكة اثيريوم المرتبطة بحساب المستخدم ويمكن استخدام هذا للوصول إلى حسابات المستخدم اثيريوم والتفاعل مع شبكة اثيريوم
      const accounts = await ethereum.request({method: "eth_requestAccounts"})
      setCurrentAccount(accounts[0])
    } catch (error) {
      console.error(error)
    }
  }

  // للمستخدم NFT يقوم بسك 1
  const mint = async () => {
    try {
      // يتم استخدام هذا للوصول إلى كائن اثيريوم والتي من تعد من الكائنات العامة
      const {ethereum} = window

      if (ethereum) {
        // يستخدم هذا للتفاعل مع البلوكتشين
        const provider = new ethers.providers.Web3Provider(ethereum, "any")
        // كائن مُوقع يتم استخدامه لمصادقة وتفويض المعاملات على البلوكتشين
        const signer = provider.getSigner()
        // يعمل هذا على اخذ مثيل للعقد بحيث نتمكن من التفاعل مع البلوكتشين
        const w3arabs = new ethers.Contract(
          contractAddress,
          contractABI,
          signer
        )

        // جلب سعر الغاز الحالي
        const gasPrices = (await provider.getGasPrice()).toString()

        // NFT من العقد الذكي وإعطاء قيمة mint استدعاء الدالة
        const w3arabsTxn = await w3arabs.mint(
          // NFT عنوان المحفظة الذي سيحصل على
          currentAccount,
          {
            // NFT تحديد قيمة
            value: ethers.utils.parseEther("0.01"),
            // حد الغاز الذي يجب إستخدامه
            gasLimit: "3000000",
            // سعر الغاز الحالي الذي يجب إستخدامه في المعامله
            gasPrice: ethers.utils.formatUnits(gasPrices, "wei")
          }
        )
        await w3arabsTxn.wait()

        window.alert("You successfully minted a W3ArabsProject!")
      } else {
        console.log("Metamask is not connected");
      }
    } catch (error) {
      console.error(error)
    }
  }

  // المقبوضة وناشر الرمز المميز NFTs جلب اجمالي
  const getTokenIdsAndOwner = async () => {
    try {
      // يتم استخدام هذا للوصول إلى كائن اثيريوم والتي من تعد من الكائنات العامة
      const {ethereum} = window

      if (ethereum) {
        // يستخدم هذا للتفاعل مع البلوكتشين
        const provider = new ethers.providers.Web3Provider(ethereum, "any")
        // يعمل هذا على اخذ مثيل للعقد بحيث نتمكن من التفاعل مع البلوكتشين
        const w3arabs = new ethers.Contract(
          contractAddress,
          contractABI,
          provider
        )

        const w3arabsIds = await w3arabs.tokenIds()
        setTokenIds(w3arabsIds.toString())

        const w3arabsOwner = await w3arabs.owner()
        setOwner(w3arabsOwner.toLowerCase())
      } else {
        console.log("Metamask is not connected");
      }
    } catch (error) {
      console.error(error)
    }
  }

  // التي يمتلكها المستخدم المتصل بالموقعNFTs جلب
  const getBalance = async () => {
    try {
      // يتم استخدام هذا للوصول إلى كائن اثيريوم والتي من تعد من الكائنات العامة
      const {ethereum} = window

      if (ethereum) {
        // يستخدم هذا للتفاعل مع البلوكتشين
        const provider = new ethers.providers.Web3Provider(ethereum, "any")
        // يعمل هذا على اخذ مثيل للعقد بحيث نتمكن من التفاعل مع البلوكتشين
        const w3arabs = new ethers.Contract(
          contractAddress,
          contractABI,
          provider
        )

        const accounts = await ethereum.request({method: "eth_requestAccounts"})
        const w3arabsBalance = await w3arabs.balanceOf(accounts[0])
        setBalance(w3arabsBalance.toString())
      } else {
        console.log("Metamask is not connected");
      }
    } catch (error) {
      console.error(error)
    }
  }

  // سحب الاموال المتواجدة في العقد الذكي
  const withdraw = async () => {
    try {
      // يتم استخدام هذا للوصول إلى كائن اثيريوم والتي من تعد من الكائنات العامة
      const {ethereum} = window

      if (ethereum) {
        // يستخدم هذا للتفاعل مع البلوكتشين
        const provider = new ethers.providers.Web3Provider(ethereum, "any")
        // كائن مُوقع يتم استخدامه لمصادقة وتفويض المعاملات على البلوكتشين
        const signer = provider.getSigner()
        // يعمل هذا على اخذ مثيل للعقد بحيث نتمكن من التفاعل مع البلوكتشين
        const w3arabs = new ethers.Contract(
          contractAddress,
          contractABI,
          signer
        )

        // owner - سحب الاموال من العقد الذكي إلى مالك العقد
        const w3arabsTxn = await w3arabs.withdraw(owner)
        await w3arabsTxn.wait()

        window.alert("You successfully withdraw a W3ArabsProject!")
      } else {
        console.log("Metamask is not connected");
      }
    } catch (error) {
      console.error(error)
    }
  }

  // تمثل المصفوفة في نهاية استدعاء الوظيفة ما هي تغييرات الحالة التي ستؤدي إلى هذا التغيير
  // في هذه الحالة كلما تغيرت قيم الوظيفتين سيتم استدعاء هذا التغيير مباشرة
  useEffect(() => {
    isConnectWallet()
    getTokenIdsAndOwner()
    getBalance()

    // قم بتعيين فاصل زمني للحصول على عدد معرّفات الرمز المميز التي يتم سكها كل 5 ثوانٍ
    setInterval(async function () {
      await getTokenIdsAndOwner()
      await getBalance()
    }, 5 * 1000)
  }, [])

  return (
    <div dir="rtl">
      <p className='text-center italic text-3xl text-rose-700 font-bold mt-10'>W3Arabs Project</p>
      {
        currentAccount ? (
          <div>
            <p className='text-center text-xl text-rose-700 font-bold mt-10'>مجموع NFTs المقبوضة: {tokenIds}</p>
            <p className='text-center text-xl text-rose-700 font-bold mt-10'>مجموع NFTs الخاصة بك: {balance}</p>
            <div className="flex justify-center">
              <button onClick={mint} className="px-4 mr-8 mt-5 py-2 text-white bg-rose-600 rounded-lg duration-150 hover:bg-rose-700 active:shadow-lg">احصل على واحدة</button>
            </div>

            {owner==currentAccount && (
              <div className="flex justify-center">
                <button onClick={withdraw} className="px-4 mr-8 mt-5 py-2 text-white bg-rose-600 rounded-lg duration-150 hover:bg-rose-700 active:shadow-lg">سحب قيمة NFTs</button>
              </div>
            )}
          </div>
        ) : (
          <div className="flex justify-center">
            <button onClick={connectWallet} className="px-4 mr-8 mt-5 py-2 text-white bg-rose-600 rounded-lg duration-150 hover:bg-rose-700 active:shadow-lg">اتصل بالمحفظة</button>
          </div>
        )
      }
    </div>
  )
}

```

<img src="https://www.web3arabs.com/courses/nfts/w3arabsnft/indexjs.png">

يعمل الكود السابق بإختصار شديد على تشغيل العقد الذكي او المشروع الذي قمنا ببناء عقده (Todo-list) في الواجهة الامامية بحيث يتمكن المستخدم من ربط محفظته واضافة المهام وتحديثها وازالتها.

```javascript
// Contract Address & ABI
const contractAddress = "add_your_smart_contract_address_here"
const contractABI = abi.abi
```

لقد قمنا أولاً بإضافة عنوان العقد الذكي الخاص بنا الذي قمنا بنشره على شبكة sepolia في المتغير (contractAddress) ومن ثم قمنا بإستدعاء ABI العقد المتواجد في مشروع Foundry في المجلد out وقمنا باستدعائه في المتغير (contractABI).


يمكنك تجربة تطبيقك الان

```bash
npm run dev
```

<img src="https://www.web3arabs.com/courses/result-front-nft.png" alt="NFT Project"/>

إنه يعمل, لقد انتهيت من بناء مشروع NFT بنجاح 🥳🥳

يمكنك الوصول الى المشروع بشكل مباشر على <a href="https://github.com/Web3Arabs/W3ArabsNFT" target="_blank"> GitHub من هنا</a>

كما هو الحال دائمًا، إذا كانت لديك أي أسئلة أو شعرت بالتعثر أو أردت فقط أن تقول مرحبًا، فقم بالإنضمام على <a href="https://discord.gg/ykgUvqMc4Q" target="_blank">Discord</a> وسنكون أكثر من سعداء لمساعدتك!
