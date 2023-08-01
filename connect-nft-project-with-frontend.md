# ربط مشروع NFT بالواجهة الامامية

بعد ان انتهينا من إنشاء مشروع NFT يمكننا تشغيله على موقعنا من اجل ان يتمكن المستخدم من التفاعل معه.

سنقوم بإستخدام منصة <a href="https://floatui.com/" target="_blank">FloatUI</a> من اجل الحصول على بعض المكونات التي ستساعدنا في بناء واجهة امامية باستخدام Reactjs و TailwindCSS بكل سهولة.

سنقوم في هذا الدرس بإستخدام إطار العمل Nextjs وحزمة create-web3-dapp والتي هي إطار عمل قائم على NextJs ومتوافق مع سلاسل الكتل الأكثر استخدامًا بما في ذلك Ethereum والتي تساعد مطوري web3 على بناء تطبيق لامركزي جاهز للإنتاج بسرعة البرق باستخدام قوالب تم بنائها سابقاً: إذهب الى مجلد المشروع وقم بكتابة هذا في terminal

```bash
npx create-web3-dapp@latest
```

اسم المشروع my-app وسنقوم بإعداد المشروع كما موضح في الصورة هنا:

<img src="https://www.web3arabs.com/courses/cw3d-settings.png"/>

> في الخطوة السابعة تم إضافة API Key الذي قمنا بإستخدامه في درس <a href="/courses/253cff7d-f8ee-4a42-8e2f-d865d2589393/lessons/4b4058b6-37cf-4893-b4ff-2cc613fa3716" target="_blank">إنشاء مشروع NFT بإستخدام IPFS</a>

يمكنك تثبيت إطار العمل TailwindCSS من خلال فتح مجلد my-app في terminal <a href="https://tailwindcss.com/docs/guides/nextjs" target="_blank">ومتابعة هذا الشرح المتواجد في وثائق Tailwind</a>

سنحتاج الان الى تثبيت ethers.js والتي ستساعدنا في التعامل مع العقد الذكي وإرسال المعاملات. قم بكتابة هذا على terminal

```bash
npm install ethers@5
```

من اجل إضافة شبكة sepolia الى التطبيق سنقوم بالذهاب الى مجلد app ثم الملف layout.jsx وجعله بهذا الشكل

```jsx
"use client";
import { WagmiConfig, createConfig, sepolia } from "wagmi";
import { ConnectKitProvider, getDefaultConfig } from "connectkit";
import Navbar from "../components/instructionsComponent/navigation/navbar";
import Footer from "../components/instructionsComponent/navigation/footer";

const chains = [sepolia]

const config = createConfig(
	getDefaultConfig({
		// Required API Keys
		alchemyId: process.env.ALCHEMY_API_KEY, // or infuraId
		walletConnectProjectId: "demo",

		// Required
		appName: "You Create Web3 Dapp",

		// Optional
		appDescription: "Your App Description",
		appUrl: "https://family.co", // your app's url
		appIcon: "https://family.co/logo.png", // your app's logo,no bigger than 1024x1024px (max. 1MB)
		chains: chains
	})
);

export default function RootLayout({ children }) {
	return (
		<html lang="en">
			<WagmiConfig config={config}>
				<ConnectKitProvider mode="dark">
					<body>
						<div
							style={{
								display: "flex",
								flexDirection: "column",
								minHeight: "105vh",
							}}
						>
							<Navbar />
							<div style={{ flexGrow: 1 }}>{children}</div>
							<Footer />
						</div>
					</body>
				</ConnectKitProvider>
			</WagmiConfig>
		</html>
	);
}
```

الان اذهب الى المجلد app ثم الملف page.jsx وقم بلصق هذا الكود ومتابعة الشرح من التعليقات المتواجدة اعلى كل سطر.

```jsx
'use client'
import "./globals.css";
import { useState, useEffect } from "react"
import { useAccount } from "wagmi"
import { ethers } from "ethers"
import abi from "../utils/W3ArabsProject.json"

export default function Home() {
  // Contract Address & ABI
  const contractAddress = "add_your_smart_contract_address_here"
  const contractABI = abi.abi

  // المتصل بالموقع address بمراقبة حالة التطبيق ما إذا كان متصل بالمحفظة او لا... بالإضافة الى  useAccount ستقوم
  const { address, connector, isConnected } = useAccount()
  // المقبوضة NFTs تخزين اجمالي عدد
  const [tokenIds, setTokenIds] = useState("0")
  // مالك العقد الذكي address تخزين
  const [owner, setOwner] = useState("")
  // التي يمتلكها الموقع NFTs تخزين اجمالي عدد
  const [balance, setBalance] = useState("0")

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

        // NFT من العقد الذكي وإعطاء قيمة mint استدعاء الدالة
        const w3arabsTxn = await w3arabs.mint({
          value: ethers.utils.parseEther("0.01"),
        })
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

        const w3arabsTxn = await w3arabs.withdraw()
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
        isConnected ? (
          <div>
            <p className='text-center text-xl text-rose-700 font-bold mt-10'>مجموع NFTs المقبوضة: {tokenIds}</p>
            <p className='text-center text-xl text-rose-700 font-bold mt-10'>مجموع NFTs الخاصة بك: {balance}</p>
            <div className="flex justify-center">
              <button onClick={mint} className="px-4 mr-8 mt-5 py-2 text-white bg-rose-600 rounded-lg duration-150 hover:bg-rose-700 active:shadow-lg">احصل على واحدة</button>
            </div>

            {owner==address.toLowerCase() && (
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

يعمل الكود السابق بإختصار شديد على تشغيل العقد الذكي او المشروع الذي قمنا ببناء عقده (Todo-list) في الواجهة الامامية بحيث يتمكن المستخدم من ربط محفظته واضافة المهام وتحديثها وازالتها.

```jsx
// Contract Address & ABI
const contractAddress = "add_your_smart_contract_address_here"
const contractABI = abi.abi
```

لقد قمنا أولاً بإضافة عنوان العقد الذكي الخاص بنا الذي قمنا بنشره على شبكة sepolia في المتغير (contractAddress) ومن ثم قمنا بإضافة ABI العقد في المجلد (utils) وقمنا باستدعائه في المتغير (contractABI).

يمكنك الحصول على ABI الخاص بعقدك الذكي من خلال فتح تطبيق "hardhat" وفتح المجلد "artifacts" ومن ثم فتح المجلد "contracts" و "Todolist.sol" وثم بنسخ الملف "Todolist.json" الى المجلد "utils" الذي قمنا بإنشائه (يمكننا فحص هذا الملف جيداً وكل ماهو مهم بالنسبة لك هو "abi" الخاص بالعقد الذكي الذي ستتعامل معه).

يمكنك تجربة تطبيقك الان

```bash
npm run dev
```
إذهب الى <a href="http://localhost:3000" target="_blank">localhost:3000</a> وقم بالإتصال بالتطبيق من خلال محفظتك وقم بإضافة بعض المهام

<img src="https://www.web3arabs.com/courses/result-cw3d-nft.png" alt="NFT Project"/>

إنه يعمل, لقد انتهيت من بناء مشروع NFT بنجاح 🥳🥳

كما هو الحال دائمًا، إذا كانت لديك أي أسئلة أو شعرت بالتعثر أو أردت فقط أن تقول مرحبًا، فقم بالإنضمام على <a href="https://t.me/Web3ArabsDAO" target="_blank">Telegram</a> او <a href="https://discord.gg/ykgUvqMc4Q" target="_blank">Discord</a> وسنكون أكثر من سعداء لمساعدتك!
