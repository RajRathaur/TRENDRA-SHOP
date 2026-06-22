<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TRENDRA SHOPKART – Online Shopping for Mobiles, Fashion, Electronics & More</title>
<!-- Firebase SDKs: these load from gstatic.com, which is blocked inside the Claude.ai
     artifact preview sandbox. This file must be downloaded and opened/hosted outside
     Claude.ai (locally, or via GitHub Pages / Netlify / Vercel / any static host)
     for Firebase to initialize correctly. -->
<script src="https://www.gstatic.com/firebasejs/12.3.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/12.3.0/firebase-firestore-compat.js"></script>
<!-- Firebase AI Logic (Gemini) — modular SDK, used by Trendra AI for real LLM-powered
     product search & chat. Modular API requires type="module"; see initTrendraAI() below
     which exposes a window-level function so the rest of the (non-module) app code can call it. -->
<script type="module" id="trendra-ai-module">
  import { initializeApp, getApps } from "https://www.gstatic.com/firebasejs/12.3.0/firebase-app.js";
  import { getAI, getGenerativeModel, GoogleAIBackend } from "https://www.gstatic.com/firebasejs/12.3.0/firebase-ai.js";
  import {
    getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword,
    sendSignInLinkToEmail, isSignInWithEmailLink, signInWithEmailLink,
    updateProfile, onAuthStateChanged
  } from "https://www.gstatic.com/firebasejs/12.3.0/firebase-auth.js";

  window.__trendraAIReady = (async function(){
    try{
      const cfg = {
        apiKey: "AIzaSyDsAc5vD41hb3ikubmFENRpLcvbn9OGrCQ",
        authDomain: "trendra-1890a.firebaseapp.com",
        projectId: "trendra-1890a",
        storageBucket: "trendra-1890a.firebasestorage.app",
        messagingSenderId: "678660854722",
        appId: "1:678660854722:web:652af74c5fce21b0a73c5b",
        measurementId: "G-DRG5YH7DNR"
      };
      // Reuse the same Firebase app instance the rest of the page creates (firebase-app-compat
      // initializes its own app under the hood, but the modular SDK needs its own handle too).
      const modularApp = getApps().length ? getApps()[0] : initializeApp(cfg);
      const ai = getAI(modularApp, { backend: new GoogleAIBackend() });
      const auth = getAuth(modularApp);

      window.__trendraAskGemini = async function(systemPrompt, history){
        // history: array of {role:'user'|'model', text:'...'}
        // Model is (re)created per-call because systemInstruction is fixed at model-creation
        // time in this SDK, and our product catalogue (passed in systemPrompt) changes live
        // as Firestore data updates — so each call must build a model with the latest catalogue.
        const model = getGenerativeModel(ai, {
          model: "gemini-3.5-flash",
          generationConfig: { responseMimeType: "application/json" },
          systemInstruction: { role: "system", parts: [{ text: systemPrompt }] }
        });
        const contents = history.map(h=>({role:h.role, parts:[{text:h.text}]}));
        const result = await model.generateContent({ contents });
        return result.response.text();
      };

      // --- Real Firebase Authentication: Email/Password + Email-link (passwordless OTP-style) ---
      window.__trendraAuth = {
        async signUp(email, password, name){
          const cred = await createUserWithEmailAndPassword(auth, email, password);
          if(name) await updateProfile(cred.user, { displayName: name });
          return { uid: cred.user.uid, email: cred.user.email, name: name || email.split('@')[0] };
        },
        async signIn(email, password){
          const cred = await signInWithEmailAndPassword(auth, email, password);
          return { uid: cred.user.uid, email: cred.user.email, name: cred.user.displayName || email.split('@')[0] };
        },
        // Sends a real sign-in link to the user's email (Firebase's free, passwordless
        // "email OTP" equivalent — no SMS cost, works on the Spark plan).
        async sendEmailOtp(email){
          const actionCodeSettings = {
            url: window.location.href.split('#')[0].split('?')[0],
            handleCodeInApp: true
          };
          await sendSignInLinkToEmail(auth, email, actionCodeSettings);
          window.localStorage.setItem('trendraEmailForSignIn', email);
        },
        // Call this on every page load to check whether the current URL is a sign-in link
        // the user clicked from their email, and complete the sign-in if so.
        async completeEmailOtpIfPresent(){
          if(!isSignInWithEmailLink(auth, window.location.href)) return null;
          let email = window.localStorage.getItem('trendraEmailForSignIn');
          if(!email){
            email = window.prompt('Confirm your email to complete sign-in:');
          }
          if(!email) return null;
          const cred = await signInWithEmailLink(auth, email, window.location.href);
          window.localStorage.removeItem('trendraEmailForSignIn');
          // Clean the magic-link params out of the URL bar without reloading the page
          window.history.replaceState({}, document.title, window.location.pathname);
          return { uid: cred.user.uid, email: cred.user.email, name: cred.user.displayName || email.split('@')[0] };
        }
      };

      return true;
    }catch(err){
      console.error("Trendra AI (Gemini) failed to initialize:", err);
      window.__trendraAskGemini = null;
      return false;
    }
  })();
</script>
<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=Poppins:wght@700;800&display=swap');
:root{
  --blue:#7c5cff;--blue-d:#6a4ce0;--yellow:#ffd60a;--orange:#ff7849;--gold:#ffb84d;
  --green:#34d399;--red:#f87171;--bg:#0c0d10;--white:#16171c;--text:#f1f1f3;
  --muted:#8b8d98;--border:#26272e;--r:8px;
  --card:#1c1d24;--card-2:#222329;
}
body.dark-mode{--bg:#0c0d10;--white:#16171c;--text:#f1f1f3;--muted:#8b8d98;--border:#26272e;}
body.dark-mode .pcard,body.dark-mode .sec,body.dark-mode .modal,body.dark-mode .cart-side,body.dark-mode .notif-panel,body.dark-mode .profile-drop{background:var(--white);}
body.dark-mode .cat-nav{background:var(--white);}
body.dark-mode .fbox,body.dark-mode .trust-bar,body.dark-mode .scard,body.dark-mode .adm-card,body.dark-mode .set-card,body.dark-mode .ch-card{background:var(--white);}
*{box-sizing:border-box;margin:0;padding:0;}
html{scroll-behavior:smooth;overflow-x:hidden;width:100%;}
body{font-family:'Inter',sans-serif;background:var(--bg);color:var(--text);font-size:14px;padding-bottom:80px;overflow-x:hidden;width:100%;}
button{font-family:inherit;cursor:pointer;border:none;outline:none;}
a{text-decoration:none;color:inherit;}
input,select,textarea{font-family:inherit;outline:none;}
.offer-strip{background:#f02;color:#fff;text-align:center;font-size:12px;font-weight:600;padding:4px 12px;letter-spacing:.3px;}
.offer-strip span{margin:0 16px;}
.hdr{background:linear-gradient(135deg,#1a1b22,#0c0d10);position:sticky;top:0;z-index:300;box-shadow:0 2px 20px rgba(0,0,0,.5);border-bottom:1px solid var(--border);}
.hdr-top{max-width:1300px;margin:0 auto;padding:10px 20px;display:flex;align-items:center;gap:18px;min-width:0;}
.logo{font-family:'Poppins',sans-serif;font-weight:800;font-size:22px;color:#fff;white-space:nowrap;cursor:pointer;line-height:1;}
.logo sub{font-size:10px;font-weight:700;color:var(--yellow);font-style:italic;display:block;letter-spacing:.5px;}
.hdr-search{flex:1;max-width:580px;display:flex;background:var(--card);border:1px solid var(--border);border-radius:var(--r);overflow:hidden;}
.hdr-search input{flex:1;border:none;padding:10px 16px;font-size:14px;color:var(--text);background:transparent;}
.hdr-search button{background:var(--blue);color:#fff;padding:0 20px;font-size:18px;border-radius:0;}
.hdr-nav{display:flex;align-items:center;gap:20px;margin-left:auto;}
.hdr-btn{background:var(--blue);color:#fff;font-weight:700;font-size:13px;padding:8px 22px;border-radius:var(--r);}
.hdr-link{color:#fff;font-weight:600;font-size:13px;cursor:pointer;display:flex;align-items:center;gap:6px;white-space:nowrap;position:relative;}
.hdr-link .badge{position:absolute;top:-8px;right:-10px;background:var(--orange);color:#fff;font-size:10px;font-weight:800;border-radius:50%;min-width:17px;height:17px;display:flex;align-items:center;justify-content:center;}
.hdr-avatar{width:30px;height:30px;border-radius:50%;background:linear-gradient(135deg,var(--blue),var(--orange));color:#fff;font-weight:800;font-size:13px;display:flex;align-items:center;justify-content:center;}
.cat-nav{background:var(--white);border-bottom:1px solid var(--border);position:sticky;top:52px;z-index:290;}
.cat-nav-inner{max-width:1300px;margin:0 auto;padding:0 20px;display:flex;gap:0;overflow-x:auto;}
.cat-nav-inner::-webkit-scrollbar{height:0;}
.cat-item{display:flex;flex-direction:column;align-items:center;gap:5px;padding:10px 20px;font-size:12px;font-weight:600;color:var(--muted);flex-shrink:0;cursor:pointer;border-bottom:2px solid transparent;transition:color .15s,border-color .15s;}
.cat-item:hover,.cat-item.active{color:var(--blue);border-bottom-color:var(--blue);}
.cat-item .ci{font-size:22px;line-height:1;}
.wrap{max-width:1300px;margin:0 auto;padding:0 20px;}
.hero-area{display:grid;grid-template-columns:170px 1fr 170px;gap:0;background:var(--white);margin-top:8px;border-radius:var(--r);border:1px solid var(--border);}
.hero-left,.hero-right{display:flex;flex-direction:column;border:1px solid var(--border);border-radius:var(--r) 0 0 var(--r);}
.hero-right{border-radius:0 var(--r) var(--r) 0;}
.side-cat{padding:10px 14px;font-size:12px;font-weight:600;color:var(--text);cursor:pointer;border-bottom:1px solid var(--border);display:flex;align-items:center;gap:8px;white-space:nowrap;transition:background .1s;}
.side-cat:hover{background:var(--card);color:var(--blue);}
.hero-banner{position:relative;overflow:hidden;min-height:300px;cursor:pointer;}
.hero-slide{position:absolute;inset:0;display:none;align-items:center;padding:40px 50px;}
.hero-slide.active{display:flex;}
.hero-slide h1{font-family:'Poppins',sans-serif;font-size:36px;font-weight:800;color:#fff;line-height:1.2;text-shadow:0 2px 8px rgba(0,0,0,.3);}
.hero-slide p{font-size:15px;color:rgba(255,255,255,.9);margin:10px 0 18px;}
.hero-slide .hbtn{background:#fff;color:#0c0d10;font-weight:800;padding:10px 24px;border-radius:var(--r);font-size:14px;display:inline-block;}
.hero-dots{position:absolute;bottom:12px;left:50%;transform:translateX(-50%);display:flex;gap:6px;}
.hero-dot{width:8px;height:8px;border-radius:50%;background:rgba(255,255,255,.5);cursor:pointer;transition:background .2s;}
.hero-dot.active{background:#fff;}
.hero-arr{position:absolute;top:50%;transform:translateY(-50%);background:rgba(255,255,255,.85);width:28px;height:64px;display:flex;align-items:center;justify-content:center;cursor:pointer;font-size:18px;z-index:2;}
.hero-arr.prev{left:0;border-radius:0 4px 4px 0;}
.hero-arr.next{right:0;border-radius:4px 0 0 4px;}
.sec{background:var(--white);border:1px solid var(--border);border-radius:var(--r);padding:16px 0;margin-top:10px;overflow:hidden;}
.sec-hdr{padding:0 16px 12px;display:flex;align-items:flex-end;justify-content:space-between;border-bottom:1px solid var(--border);}
.sec-hdr h2{font-size:19px;font-weight:800;font-family:'Poppins',sans-serif;color:var(--text);}
.sec-hdr .sub{font-size:12px;color:var(--muted);margin-top:2px;}
.sec-hdr .view-all{background:var(--card);color:var(--blue);border:1px solid var(--blue);padding:8px 22px;border-radius:var(--r);font-weight:700;font-size:13px;cursor:pointer;}
.sec-hdr .timer{background:var(--card-2);color:#fff;padding:6px 14px;border-radius:20px;font-size:12px;font-weight:700;letter-spacing:1px;border:1px solid var(--border);}
.sec-hdr .timer b{color:var(--yellow);}
.prod-scroll{display:flex;gap:0;overflow-x:auto;padding:0;}
.prod-scroll::-webkit-scrollbar{height:4px;}
.prod-scroll::-webkit-scrollbar-thumb{background:var(--border);}
.prod-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));padding:10px 16px;gap:0;}
.pcard{border-right:1px solid var(--border);border-bottom:1px solid var(--border);padding:16px 10px 12px;cursor:pointer;transition:box-shadow .2s,transform .15s;position:relative;background:var(--white);text-align:center;min-width:170px;}
.pcard:hover{box-shadow:0 8px 28px rgba(124,92,255,.18);z-index:2;transform:translateY(-2px);}
.pcard .wish{position:absolute;top:8px;right:8px;font-size:16px;background:none;padding:2px;}
.pcard .img-wrap{height:150px;display:flex;align-items:center;justify-content:center;font-size:72px;margin-bottom:10px;border-radius:var(--r);overflow:hidden;}
.pcard .pname{font-size:13px;font-weight:500;color:var(--text);line-height:1.35;margin-bottom:4px;text-align:left;}
.pcard .pname-short{font-size:13px;font-weight:600;margin-bottom:2px;text-align:left;color:var(--text);}
.pcard .prating{display:flex;align-items:center;gap:5px;margin-bottom:6px;}
.rating-star{background:var(--green);color:#06281c;font-size:11px;font-weight:700;padding:2px 6px;border-radius:3px;display:inline-flex;align-items:center;gap:2px;}
.rating-count{font-size:11px;color:var(--muted);}
.pcard .pprice{display:flex;align-items:baseline;gap:7px;flex-wrap:wrap;}
.pprice .sp{font-size:17px;font-weight:700;color:var(--text);}
.pprice .mrp{font-size:12px;color:var(--muted);text-decoration:line-through;}
.pprice .off{font-size:12px;color:var(--green);font-weight:700;}
.pcard .add-btn{width:100%;margin-top:10px;background:linear-gradient(135deg,var(--orange),#ff5a5f);color:#fff;font-weight:700;font-size:12px;padding:8px;border-radius:var(--r);letter-spacing:.4px;text-transform:uppercase;transition:filter .15s;}
.pcard .add-btn:hover{filter:brightness(1.12);}
.pcard .add-btn.done{background:var(--green);color:#06281c;}
.pcard .badge-new{position:absolute;top:8px;left:8px;background:var(--blue);color:#fff;font-size:10px;font-weight:800;padding:2px 7px;border-radius:2px;}
.deal-strip{display:grid;grid-template-columns:repeat(auto-fill,minmax(120px,1fr));gap:0;padding:10px 0;}
.deal-thumb{display:flex;flex-direction:column;align-items:center;gap:6px;padding:14px 8px;border-right:1px solid var(--border);cursor:pointer;transition:background .15s;}
.deal-thumb:hover{background:var(--card);}
.deal-thumb .dt-img{font-size:44px;}
.deal-thumb .dt-off{font-size:13px;font-weight:700;color:var(--orange);}
.deal-thumb .dt-cat{font-size:11px;color:var(--muted);}
.banner-strip{display:grid;grid-template-columns:repeat(4,1fr);gap:10px;margin-top:10px;}
.mini-banner{border-radius:var(--r);padding:20px 16px;min-height:120px;display:flex;flex-direction:column;justify-content:center;cursor:pointer;position:relative;overflow:hidden;color:#fff;}
.mini-banner h3{font-size:16px;font-weight:800;margin-bottom:4px;}
.mini-banner p{font-size:12px;opacity:.9;}
.mini-banner span{font-size:11px;font-weight:700;background:rgba(255,255,255,.25);padding:3px 10px;border-radius:2px;margin-top:8px;display:inline-block;}
.feature-row{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:10px;margin-top:10px;}
.fbox{background:var(--white);border-radius:var(--r);padding:20px;text-align:center;border:1px solid var(--border);}
.fbox .fi{font-size:32px;margin-bottom:8px;}
.fbox h4{font-size:13px;font-weight:700;margin-bottom:4px;color:var(--text);}
.fbox p{font-size:12px;color:var(--muted);}
.trust-bar{background:var(--white);border-top:1px solid var(--border);padding:14px 0;margin-top:10px;}
.trust-inner{max-width:1300px;margin:0 auto;padding:0 20px;display:flex;justify-content:space-around;flex-wrap:wrap;gap:12px;}
.trust-item{display:flex;align-items:center;gap:10px;font-size:13px;color:var(--text);}
.trust-icon{font-size:24px;}
.trust-info b{display:block;font-weight:700;font-size:13px;}
.trust-info span{font-size:11px;color:var(--muted);}
.footer{background:#08090b;color:#8b8d98;margin-top:20px;}
.footer-top{max-width:1300px;margin:0 auto;padding:40px 20px;display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:30px;}
.footer-col h5{color:#fff;font-size:12px;font-weight:700;letter-spacing:.8px;text-transform:uppercase;margin-bottom:14px;}
.footer-col a{display:block;font-size:12px;padding:4px 0;cursor:pointer;transition:color .15s;}
.footer-col a:hover{color:#fff;}
.footer-bottom{border-top:1px solid rgba(255,255,255,.1);padding:16px 20px;text-align:center;font-size:12px;}
.footer-bottom .upi{color:var(--yellow);font-weight:700;}
.overlay{position:fixed;inset:0;background:rgba(0,0,0,.55);z-index:400;opacity:0;visibility:hidden;transition:opacity .2s;}
.overlay.on{opacity:1;visibility:visible;}
.modal{position:fixed;top:50%;left:50%;transform:translate(-50%,-50%) scale(.95);background:var(--white);border:1px solid var(--border);color:var(--text);border-radius:10px;z-index:410;max-width:92vw;max-height:90vh;overflow-y:auto;opacity:0;visibility:hidden;transition:opacity .2s,transform .2s;}
.modal.on{opacity:1;visibility:visible;transform:translate(-50%,-50%) scale(1);}
.mclose{position:absolute;top:12px;right:14px;background:var(--card);color:var(--text);border-radius:50%;width:28px;height:28px;font-size:15px;}
.cart-side{position:fixed;top:0;right:0;bottom:0;width:390px;max-width:94vw;background:var(--white);color:var(--text);z-index:410;transform:translateX(100%);transition:transform .3s;display:flex;flex-direction:column;box-shadow:-4px 0 30px rgba(0,0,0,.5);}
.cart-side.on{transform:translateX(0);}
.cart-hdr{background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;padding:16px 18px;display:flex;justify-content:space-between;align-items:center;font-weight:700;font-size:16px;}
.cart-hdr button{background:none;color:#fff;font-size:20px;}
.cart-body{flex:1;overflow-y:auto;padding:10px 14px;}
.cart-empty{text-align:center;padding:60px 20px;color:var(--muted);}
.cart-empty .ce-icon{font-size:56px;margin-bottom:14px;}
.cart-item{display:flex;gap:12px;padding:14px 0;border-bottom:1px solid var(--border);}
.ci-img{width:64px;height:64px;border-radius:var(--r);display:flex;align-items:center;justify-content:center;font-size:30px;flex-shrink:0;}
.ci-info{flex:1;}
.ci-name{font-size:13px;font-weight:500;line-height:1.3;margin-bottom:5px;color:var(--text);}
.ci-price{font-weight:700;font-size:14px;color:var(--text);}
.ci-qty{display:flex;align-items:center;gap:10px;margin-top:6px;}
.ci-qty button{width:24px;height:24px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:3px;font-weight:700;}
.ci-remove{background:none;color:var(--orange);font-size:12px;font-weight:700;margin-top:4px;}
.cart-foot{border-top:1px solid var(--border);padding:14px 18px;}
.sum-row{display:flex;justify-content:space-between;font-size:13px;margin-bottom:7px;color:var(--muted);}
.sum-row.total{color:var(--text);font-weight:800;font-size:16px;border-top:1px dashed var(--border);padding-top:8px;margin-top:5px;}
.coupon-strip{background:rgba(52,211,153,.12);color:var(--green);font-size:12px;font-weight:600;padding:8px 10px;border-radius:3px;margin-bottom:8px;display:flex;justify-content:space-between;}
.cart-cta{width:100%;background:linear-gradient(135deg,var(--gold),var(--orange));color:#1a0e00;font-weight:800;font-size:14px;padding:12px;border-radius:var(--r);text-transform:uppercase;margin-top:8px;}
.login-modal{width:360px;padding:30px;}
.login-modal h2{font-size:19px;font-weight:800;margin-bottom:4px;}
.login-modal .lsub{font-size:13px;color:var(--muted);margin-bottom:16px;}
.l-tabs{display:flex;gap:8px;margin-bottom:16px;}
.l-tabs button{flex:1;padding:9px;border-radius:3px;font-weight:700;font-size:13px;background:var(--card);color:var(--text);}
.l-tabs button.act{background:var(--blue);color:#fff;}
.lfield{margin-bottom:12px;}
.lfield label{font-size:11px;font-weight:700;color:var(--muted);display:block;margin-bottom:4px;text-transform:uppercase;letter-spacing:.5px;}
.lfield input{width:100%;padding:10px 12px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:3px;font-size:14px;}
.lfield input:focus{border-color:var(--blue);}
.l-submit{width:100%;margin-top:14px;background:linear-gradient(135deg,var(--orange),#ff5a5f);color:#fff;font-weight:800;padding:12px;border-radius:3px;font-size:14px;text-transform:uppercase;}
.l-admin-hint{text-align:center;margin-top:10px;font-size:12px;color:var(--muted);}
.l-admin-hint b{color:var(--blue);cursor:pointer;}
.l-divider{display:flex;align-items:center;gap:10px;margin:12px 0;color:var(--muted);font-size:12px;}
.l-divider::before,.l-divider::after{content:"";flex:1;height:1px;background:var(--border);}
.social-btns{display:flex;gap:8px;}
.social-btn{flex:1;padding:9px;border:1px solid var(--border);border-radius:3px;font-size:13px;font-weight:600;background:var(--card);color:var(--text);display:flex;align-items:center;justify-content:center;gap:6px;}
.otp-choice{display:flex;gap:8px;margin-top:2px;}
.otp-choice-btn{flex:1;padding:9px 8px;border:1px solid var(--border);border-radius:3px;font-size:12px;font-weight:600;background:var(--card);color:var(--muted);cursor:pointer;display:flex;flex-direction:column;align-items:center;gap:2px;}
.otp-choice-btn.act{border-color:var(--blue);color:var(--blue);background:rgba(40,116,240,.08);}
.otp-choice-btn:disabled{opacity:.55;cursor:not-allowed;}
.otp-soon{font-size:9px;font-weight:700;color:var(--orange);text-transform:uppercase;}
.otp-hint{font-size:11px;color:var(--muted);margin-top:10px;line-height:1.5;}
.pd-modal{width:860px;display:flex;max-height:88vh;}
.pd-imgs{width:38%;background:var(--card);display:flex;align-items:center;justify-content:center;font-size:120px;flex-shrink:0;border-radius:8px 0 0 8px;}
.pd-info{flex:1;padding:24px;overflow-y:auto;}
.pd-info h2{font-size:18px;font-weight:700;line-height:1.4;margin-bottom:8px;color:var(--text);}
.pd-meta{display:flex;align-items:center;gap:12px;margin-bottom:12px;flex-wrap:wrap;}
.pd-price-row{display:flex;align-items:baseline;gap:10px;margin:10px 0;}
.pd-price{font-size:26px;font-weight:800;color:var(--text);}
.pd-mrp{font-size:15px;color:var(--muted);text-decoration:line-through;}
.pd-off{font-size:16px;color:var(--green);font-weight:700;}
.pd-emi{font-size:12px;color:var(--muted);margin-bottom:12px;}
.pd-spec{margin:14px 0;}
.pd-spec h4{font-size:13px;font-weight:700;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;margin-bottom:8px;}
.pd-spec table{font-size:13px;width:100%;border-collapse:collapse;color:var(--text);}
.pd-spec table td{padding:5px 0;vertical-align:top;border-bottom:1px solid var(--border);}
.pd-spec table td:first-child{color:var(--muted);width:40%;padding-right:10px;}
.pd-actions{display:flex;gap:10px;margin-top:16px;}
.pd-actions button{flex:1;padding:13px;font-weight:800;font-size:13px;border-radius:var(--r);text-transform:uppercase;letter-spacing:.4px;}
.btn-cart{background:var(--orange);color:#fff;}
.btn-buy{background:var(--gold);color:#fff;}
.pd-badges{display:flex;gap:8px;flex-wrap:wrap;margin:10px 0;}
.pd-badge{background:#e8f5e9;color:var(--green);font-size:11px;font-weight:700;padding:3px 9px;border-radius:2px;}
/* PRODUCT DETAIL PAGE (full page, Flipkart-style) */
.pdp-crumb{font-size:12px;color:var(--muted);padding:10px 0;}
.pdp-crumb span{cursor:pointer;}
.pdp-crumb span:hover{color:var(--blue);}
.pdp-wrap{display:grid;grid-template-columns:420px 1fr;gap:24px;background:#fff;border-radius:var(--r);padding:24px;}
.pdp-left{position:sticky;top:140px;align-self:start;}
.pdp-right{min-width:0;}
.pdp-img{width:100%;height:380px;border-radius:var(--r);display:flex;align-items:center;justify-content:center;font-size:140px;overflow:hidden;margin-bottom:14px;}
.pdp-img img{width:100%;height:100%;object-fit:cover;}
.pdp-actions{display:flex;gap:10px;}
.pdp-actions button{flex:1;padding:14px;font-weight:800;font-size:14px;border-radius:var(--r);text-transform:uppercase;letter-spacing:.4px;}
.pdp-title{font-size:22px;font-weight:700;line-height:1.4;margin-bottom:6px;}
.pdp-brand{font-size:13px;color:var(--muted);margin-bottom:10px;}
.pdp-rating-row{display:flex;align-items:center;gap:12px;margin-bottom:14px;}
.pdp-price-row{display:flex;align-items:baseline;gap:12px;margin:14px 0;}
.pdp-price{font-size:32px;font-weight:800;}
.pdp-mrp{font-size:17px;color:var(--muted);text-decoration:line-through;}
.pdp-off{font-size:18px;color:var(--green);font-weight:700;}
.pdp-section{margin:22px 0;padding-top:18px;border-top:1px solid #f5f5f5;}
.pdp-section h3{font-size:15px;font-weight:700;margin-bottom:12px;text-transform:uppercase;letter-spacing:.4px;color:var(--text);}
.pdp-spec-table{width:100%;font-size:13px;border-collapse:collapse;}
.pdp-spec-table td{padding:8px 0;border-bottom:1px solid #f5f5f5;}
.pdp-spec-table td:first-child{color:var(--muted);width:35%;}
.pdp-rating-summary{display:flex;gap:30px;align-items:center;margin-bottom:18px;}
.pdp-rating-big{text-align:center;}
.pdp-rating-num{font-size:42px;font-weight:800;}
.pdp-rating-bars{flex:1;}
.pdp-rbar-row{display:flex;align-items:center;gap:8px;font-size:12px;margin-bottom:5px;}
.pdp-rbar-track{flex:1;height:6px;background:#f0f0f0;border-radius:3px;overflow:hidden;}
.pdp-rbar-fill{height:100%;background:var(--green);}
.pdp-review{border-bottom:1px solid #f5f5f5;padding:16px 0;}
.pdp-review-head{display:flex;align-items:center;gap:10px;margin-bottom:6px;}
.pdp-review-name{font-weight:700;font-size:13px;}
.pdp-review-date{font-size:11px;color:var(--muted);}
.pdp-review-text{font-size:13px;color:var(--text);line-height:1.6;}
.pdp-related{display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:0;}
@media(max-width:900px){.pdp-wrap{grid-template-columns:1fr;}.pdp-left{position:static;}.pdp-img{height:260px;font-size:90px;}}
.spin-modal{width:420px;padding:26px;text-align:center;}
.spin-modal h2{font-size:20px;font-weight:800;margin-bottom:4px;}
.spin-modal .ssub{font-size:13px;color:var(--muted);margin-bottom:16px;}
.wheel-wrap{position:relative;width:260px;height:260px;margin:0 auto 16px;}
.wheel{width:260px;height:260px;border-radius:50%;border:8px solid #fff;box-shadow:0 0 0 4px var(--blue),0 10px 24px rgba(0,0,0,.2);background:conic-gradient(from 0deg,#2874f0 0 45deg,#9e9e9e 45deg 90deg,#fb641b 90deg 135deg,#388e3c 135deg 180deg,#ff9f00 180deg 225deg,#e91e63 225deg 270deg,#ffe11b 270deg 315deg,#757575 315deg 360deg);transition:transform 4s cubic-bezier(.17,.67,.16,.99);}
.wheel-ptr{position:absolute;top:-8px;left:50%;transform:translateX(-50%);width:0;height:0;border-left:12px solid transparent;border-right:12px solid transparent;border-top:22px solid #212121;z-index:5;}
.wheel-center{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);width:50px;height:50px;border-radius:50%;background:#fff;display:flex;align-items:center;justify-content:center;font-size:22px;z-index:4;}
.spin-btn{background:var(--blue);color:#fff;font-weight:800;font-size:14px;padding:12px 30px;border-radius:30px;text-transform:uppercase;}
.spin-btn:disabled{background:#bdbdbd;cursor:not-allowed;}
.spin-res{margin-top:14px;background:#e8f5e9;color:var(--green);font-weight:700;padding:12px;border-radius:5px;display:none;}
.spin-res.on{display:block;}
.spin-code{background:#fff;border:1px dashed var(--green);padding:4px 12px;border-radius:3px;font-family:monospace;font-size:14px;letter-spacing:1px;display:inline-block;margin:6px 0;}
.coupon-apply-btn{background:var(--green);color:#fff;font-weight:700;font-size:12px;padding:8px 18px;border-radius:20px;margin-top:8px;}
.co-modal{width:580px;padding:0;overflow:hidden;}
/* BARISTA AI */
.barista-modal{width:480px;padding:0;display:flex;flex-direction:column;max-height:82vh;}
.bar-hdr{display:flex;align-items:center;gap:12px;padding:20px 24px 14px;border-bottom:1px solid var(--border);}
.bar-avatar{width:44px;height:44px;border-radius:50%;background:linear-gradient(135deg,#6f4e37,#c08552);display:flex;align-items:center;justify-content:center;font-size:22px;flex-shrink:0;}
.bar-hdr h2{font-size:17px;font-weight:800;}
.bar-hdr .bar-sub{font-size:12px;color:var(--muted);margin-top:2px;}
.bar-chat{flex:1;overflow-y:auto;padding:16px 20px;min-height:240px;max-height:360px;display:flex;flex-direction:column;gap:10px;}
.bar-msg{max-width:82%;padding:10px 14px;border-radius:14px;font-size:13px;line-height:1.5;}
.bar-msg.bot{background:var(--card);color:var(--text);align-self:flex-start;border-bottom-left-radius:4px;}
.bar-msg.user{background:linear-gradient(135deg,#6f4e37,#c08552);color:#fff;align-self:flex-end;border-bottom-right-radius:4px;}
.bar-msg .bm-actions{display:flex;flex-wrap:wrap;gap:6px;margin-top:8px;}
.bar-msg .bm-chip{background:rgba(255,255,255,.08);border:1px solid var(--border);color:var(--text);font-size:11px;font-weight:700;padding:6px 12px;border-radius:14px;cursor:pointer;}
.bar-msg.bot .bm-chip:hover{border-color:#c08552;}
.bar-msg .bm-addbtn{background:var(--green);color:#06281c;font-size:12px;font-weight:800;padding:7px 14px;border-radius:8px;margin-top:8px;}
.bar-input-row{display:flex;gap:8px;padding:14px 16px;border-top:1px solid var(--border);flex-shrink:0;}
.bar-input{flex:1;padding:11px 14px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:24px;font-size:13px;}
.bar-input:focus{border-color:#c08552;}
.bar-send{width:40px;height:40px;border-radius:50%;background:linear-gradient(135deg,#6f4e37,#c08552);color:#fff;font-size:16px;flex-shrink:0;}
@media(max-width:600px){.barista-modal{width:96vw;max-height:88vh;}}
.co-steps{display:flex;align-items:center;background:#fafafa;border-bottom:1px solid var(--border);padding:14px 24px;}
.co-step{flex:1;text-align:center;font-size:11px;font-weight:700;color:var(--muted);}
.co-step.act{color:var(--blue);}
.co-step.done{color:var(--green);}
.co-step-num{width:22px;height:22px;border-radius:50%;border:2px solid var(--border);background:#fff;display:flex;align-items:center;justify-content:center;margin:0 auto 4px;font-size:11px;font-weight:800;}
.co-step.act .co-step-num{border-color:var(--blue);background:var(--blue);color:#fff;}
.co-step.done .co-step-num{border-color:var(--green);background:var(--green);color:#fff;}
.co-line{flex:1;height:2px;background:var(--border);margin-top:-11px;}
.co-line.done{background:var(--green);}
.co-body{padding:18px 24px;}
.co-label{font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:.5px;color:var(--muted);display:block;margin:10px 0 4px;}
.co-input{width:100%;padding:10px 12px;border:1px solid var(--border);border-radius:3px;font-size:13px;}
.co-input:focus{border-color:var(--blue);}
.co-2col{display:grid;grid-template-columns:1fr 1fr;gap:10px;}
.pm-opts{display:flex;gap:8px;flex-wrap:wrap;margin:10px 0;}
.pm-opt{flex:1;min-width:90px;border:2px solid var(--border);border-radius:5px;padding:10px 8px;text-align:center;cursor:pointer;font-weight:700;font-size:11px;transition:border-color .15s,background .15s;}
.pm-opt.act{border-color:var(--blue);background:#e8f0fe;color:var(--blue);}
.pm-opt .pmi{font-size:20px;display:block;margin-bottom:3px;}
.upi-panel{background:#f9f9f9;border:1px solid var(--border);border-radius:5px;padding:16px;text-align:center;margin-top:10px;display:none;}
.upi-panel.on{display:block;}
.upi-id-box{font-family:monospace;font-weight:800;font-size:15px;background:#fff;padding:9px 16px;border-radius:3px;display:inline-block;border:2px dashed var(--blue);color:var(--blue);letter-spacing:.5px;margin:8px 0;}
.upi-apps{display:flex;gap:8px;justify-content:center;margin:10px 0;}
.upi-app{background:#fff;border:1px solid var(--border);border-radius:5px;padding:7px 14px;font-size:12px;font-weight:700;cursor:pointer;}
.upi-app:hover{background:#e8f0fe;}
.card-panel,.nb-panel,.cod-panel{display:none;margin-top:10px;}
.card-panel.on,.nb-panel.on,.cod-panel.on{display:block;}
.cod-notice{background:#fff8e1;border:1px solid #ffe082;border-radius:5px;padding:12px;font-size:13px;line-height:1.6;}
.co-next{width:100%;background:var(--gold);color:#fff;font-weight:800;font-size:14px;padding:12px;border-radius:3px;text-transform:uppercase;margin-top:14px;}
.co-next.blue{background:var(--blue);}
.order-ok{width:400px;padding:36px;text-align:center;}
.order-ok .ok-icon{font-size:58px;margin-bottom:10px;}
.order-ok h2{font-size:21px;font-weight:800;margin-bottom:8px;}
.order-ok .osub{font-size:13px;color:var(--muted);line-height:1.6;}
.order-id{display:inline-block;margin:14px 0;background:var(--card);padding:9px 20px;border-radius:20px;font-weight:800;font-family:monospace;letter-spacing:.5px;color:var(--text);}
.order-ok .cont-btn{width:100%;background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;font-weight:800;padding:12px;border-radius:3px;margin-top:10px;font-size:14px;text-transform:uppercase;}
.wish-modal{width:680px;padding:24px;}
.wish-modal h2{font-size:18px;font-weight:800;margin-bottom:16px;}
.wish-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(150px,1fr));gap:12px;}
.orders-modal{width:640px;padding:24px;}
.orders-modal h2{font-size:18px;font-weight:800;margin-bottom:16px;}
.order-card{border:1px solid var(--border);border-radius:5px;padding:14px;margin-bottom:12px;background:var(--card);}
.order-card .oc-head{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px;}
.order-card .oc-id{font-weight:800;font-size:13px;color:var(--text);}
.order-card .oc-status{font-size:11px;font-weight:700;padding:3px 10px;border-radius:12px;}
.st-delivered{background:rgba(52,211,153,.15);color:var(--green);}
.st-shipped{background:rgba(124,92,255,.15);color:var(--blue);}
.st-pending{background:rgba(255,184,77,.15);color:var(--gold);}
.st-cancelled{background:rgba(248,113,113,.15);color:var(--red);}
.oc-items{font-size:13px;color:var(--muted);}
.oc-total{font-weight:700;font-size:14px;margin-top:6px;color:var(--text);}
.notif-panel{position:fixed;top:52px;right:20px;width:320px;background:var(--white);border-radius:8px;box-shadow:0 8px 40px rgba(0,0,0,.5);z-index:350;display:none;border:1px solid var(--border);}
.notif-panel.on{display:block;}
.notif-hdr{padding:12px 16px;border-bottom:1px solid var(--border);font-weight:700;font-size:14px;display:flex;justify-content:space-between;color:var(--text);}
.notif-item{padding:12px 16px;border-bottom:1px solid var(--border);display:flex;gap:10px;cursor:pointer;font-size:13px;color:var(--text);}
.notif-item:hover{background:var(--card);}
.notif-item.unread{background:rgba(124,92,255,.08);}
.notif-dot{width:8px;height:8px;border-radius:50%;background:var(--blue);flex-shrink:0;margin-top:4px;}
.profile-drop{position:fixed;top:52px;right:120px;width:240px;background:var(--white);border-radius:8px;box-shadow:0 8px 40px rgba(0,0,0,.5);z-index:350;display:none;border:1px solid var(--border);}
.profile-drop.on{display:block;}
.profile-drop .pd-user{padding:14px 16px;border-bottom:1px solid var(--border);display:flex;align-items:center;gap:10px;}
.profile-drop a{display:block;padding:11px 16px;font-size:13px;font-weight:500;cursor:pointer;border-bottom:1px solid var(--border);color:var(--text);display:flex;align-items:center;gap:10px;}
.profile-drop a:hover{background:var(--card);}
.pf-modal{width:560px;padding:26px;}
.pf-modal h2{font-size:18px;font-weight:800;margin-bottom:16px;}
.pf-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px;}
.pf-full{grid-column:1/-1;}
.pf-label{font-size:11px;font-weight:700;color:var(--muted);display:block;margin-bottom:4px;text-transform:uppercase;letter-spacing:.4px;}
.pf-input{width:100%;padding:9px 11px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:3px;font-size:13px;}
.pf-input:focus{border-color:var(--blue);}
.pf-actions{display:flex;gap:10px;margin-top:16px;}
.pf-actions button{flex:1;padding:11px;font-weight:800;border-radius:3px;font-size:13px;text-transform:uppercase;}
.pf-save{background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;}
.pf-cancel{background:var(--card);color:var(--text);}
.toast{position:fixed;bottom:24px;left:50%;transform:translateX(-50%) translateY(30px);background:var(--card-2);color:var(--text);border:1px solid var(--border);padding:11px 22px;border-radius:24px;font-size:13px;font-weight:600;z-index:600;opacity:0;transition:opacity .25s,transform .25s;box-shadow:0 6px 24px rgba(0,0,0,.5);white-space:nowrap;}
.toast.on{opacity:1;transform:translateX(-50%) translateY(0);}
.spin-fab{position:fixed;right:20px;bottom:24px;background:linear-gradient(135deg,var(--gold),var(--orange));color:#1a0e00;font-weight:800;font-size:13px;padding:12px 18px;border-radius:50px;box-shadow:0 6px 24px rgba(255,120,73,.4);z-index:240;animation:pulse 2.5s infinite;}
@keyframes pulse{0%,100%{transform:scale(1);}50%{transform:scale(1.05);}}
.btt{position:fixed;right:20px;bottom:80px;background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;width:38px;height:38px;border-radius:50%;font-size:18px;z-index:240;display:none;align-items:center;justify-content:center;box-shadow:0 4px 16px rgba(0,0,0,.5);}
.btt.on{display:flex;}
.adm{position:fixed;inset:0;background:var(--bg);z-index:500;display:none;flex-direction:column;}
.adm.on{display:flex;}
.adm-top{background:#08090b;color:#fff;display:flex;align-items:center;gap:14px;padding:0 20px;height:54px;flex-shrink:0;border-bottom:1px solid var(--border);}
.adm-brand{font-family:'Poppins',sans-serif;font-weight:800;font-size:17px;}
.adm-brand span{color:var(--yellow);}
.adm-top-btn{background:rgba(255,255,255,.08);color:#fff;padding:7px 14px;border-radius:3px;font-weight:600;font-size:12px;}
.adm-top-btn:hover{background:rgba(255,255,255,.16);}
.adm-top-btn.danger{background:#7f1d1d;}
.adm-body{display:flex;flex:1;overflow:hidden;}
.adm-side{width:200px;background:#08090b;color:#8b8d98;flex-shrink:0;overflow-y:auto;padding:14px 8px;border-right:1px solid var(--border);}
.adm-side .ns{font-size:10px;font-weight:700;text-transform:uppercase;letter-spacing:.8px;color:#5a5c66;padding:10px 10px 5px;}
.adm-nav{width:100%;text-align:left;padding:9px 12px;border-radius:5px;font-weight:600;font-size:13px;background:none;color:#8b8d98;display:flex;align-items:center;gap:9px;margin-bottom:2px;}
.adm-nav:hover{background:rgba(255,255,255,.06);color:#fff;}
.adm-nav.on{background:var(--blue);color:#fff;}
.adm-content{flex:1;overflow-y:auto;padding:22px;}
.adm-page{display:none;}
.adm-page.on{display:block;}
.adm-title{font-family:'Poppins',sans-serif;font-size:20px;font-weight:800;margin-bottom:18px;display:flex;align-items:center;justify-content:space-between;color:var(--text);}
.stat-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(190px,1fr));gap:14px;margin-bottom:20px;}
.scard{background:var(--white);border-radius:5px;padding:16px;border:1px solid var(--border);display:flex;align-items:center;gap:12px;}
.sc-icon{width:46px;height:46px;border-radius:8px;display:flex;align-items:center;justify-content:center;font-size:20px;flex-shrink:0;}
.sc-info .lbl{font-size:11px;color:var(--muted);font-weight:700;text-transform:uppercase;letter-spacing:.5px;}
.sc-info .val{font-size:22px;font-weight:800;margin-top:2px;color:var(--text);}
.sc-info .chg{font-size:11px;font-weight:700;margin-top:2px;}
.chg-up{color:var(--green);}
.chg-dn{color:var(--red);}
.adm-card{background:var(--white);border-radius:5px;border:1px solid var(--border);overflow:hidden;margin-bottom:18px;}
.adm-card-hdr{padding:13px 16px;border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;gap:10px;flex-wrap:wrap;}
.adm-card-title{font-size:14px;font-weight:700;color:var(--text);}
.adm-tbl-wrap{overflow-x:auto;}
table.atbl{width:100%;border-collapse:collapse;font-size:13px;min-width:580px;color:var(--text);}
table.atbl th,table.atbl td{padding:10px 14px;text-align:left;border-bottom:1px solid var(--border);vertical-align:middle;}
table.atbl th{background:var(--card);font-weight:700;font-size:11px;text-transform:uppercase;letter-spacing:.5px;color:var(--muted);}
table.atbl tr:last-child td{border-bottom:none;}
table.atbl tr:hover td{background:var(--card);}
.sbadge{padding:3px 9px;border-radius:11px;font-size:11px;font-weight:700;text-transform:uppercase;white-space:nowrap;}
.sb-pending{background:rgba(255,184,77,.15);color:var(--gold);}
.sb-paid{background:rgba(52,211,153,.15);color:var(--green);}
.sb-shipped{background:rgba(124,92,255,.15);color:var(--blue);}
.sb-delivered{background:rgba(52,211,153,.15);color:var(--green);}
.sb-cancelled{background:rgba(248,113,113,.15);color:var(--red);}
.sb-upi{background:rgba(124,92,255,.15);color:var(--blue);}
.sb-cod{background:rgba(255,118,234,.15);color:#ff76ea;}
.sb-card{background:rgba(124,92,255,.15);color:var(--blue);}
.sb-low{background:rgba(248,113,113,.15);color:var(--red);}
.sb-ok{background:rgba(52,211,153,.15);color:var(--green);}
.adm-srch{padding:8px 12px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:3px;font-size:13px;width:220px;}
.adm-srch:focus{border-color:var(--blue);}
.adm-btn{background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;padding:8px 14px;border-radius:3px;font-weight:700;font-size:12px;}
.adm-btn:hover{filter:brightness(1.1);}
.sel-sm{padding:6px 10px;border-radius:3px;border:1px solid var(--border);background:var(--card);color:var(--text);font-size:12px;font-weight:600;font-family:inherit;}
.ico-btn{width:28px;height:28px;border-radius:3px;font-size:13px;background:var(--card);color:var(--text);}
.ico-btn:hover{background:var(--card-2);}
.ico-btn.del:hover{background:rgba(248,113,113,.15);color:var(--red);}
.chart-grid{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-bottom:18px;}
.ch-card{background:var(--white);border-radius:5px;border:1px solid var(--border);padding:16px;}
.ch-card h3{font-size:13px;font-weight:700;margin-bottom:12px;color:var(--text);}
.bar-chart{display:flex;align-items:flex-end;gap:6px;height:110px;}
.bgrp{display:flex;flex-direction:column;align-items:center;flex:1;gap:3px;}
.bar{background:linear-gradient(180deg,var(--blue),var(--blue-d));border-radius:3px 3px 0 0;width:100%;transition:height .5s;}
.blbl{font-size:10px;color:var(--muted);font-weight:600;}
.bval{font-size:10px;font-weight:700;color:var(--text);}
.donut-wr{display:flex;align-items:center;gap:14px;}
.dleg{display:flex;flex-direction:column;gap:7px;}
.dleg-item{display:flex;align-items:center;gap:7px;font-size:12px;color:var(--text);}
.dleg-dot{width:9px;height:9px;border-radius:50%;flex-shrink:0;}
.set-grid{display:grid;grid-template-columns:1fr 1fr;gap:14px;}
.set-card{background:var(--white);border-radius:5px;border:1px solid var(--border);padding:18px;}
.set-card h3{font-size:13px;font-weight:700;margin-bottom:12px;color:var(--text);}
.set-row{display:flex;align-items:center;justify-content:space-between;padding:9px 0;border-bottom:1px solid var(--border);font-size:13px;color:var(--text);}
.set-row:last-child{border-bottom:none;}
.set-input{width:160px;padding:6px 10px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:3px;font-size:13px;}
.toggle-wrap{position:relative;width:38px;height:20px;display:inline-block;}
.toggle-wrap input{opacity:0;width:0;height:0;}
.tog-sl{position:absolute;inset:0;background:#3a3b42;border-radius:20px;cursor:pointer;transition:.3s;}
.tog-sl::before{content:"";position:absolute;height:14px;width:14px;left:3px;bottom:3px;background:#fff;border-radius:50%;transition:.3s;}
.toggle-wrap input:checked+.tog-sl{background:var(--green);}
.toggle-wrap input:checked+.tog-sl::before{transform:translateX(18px);}
.bg-mob{background:linear-gradient(135deg,#667eea,#764ba2);}
.bg-elec{background:linear-gradient(135deg,#36d1dc,#5b86e5);}
.bg-fash{background:linear-gradient(135deg,#ff9a9e,#fad0c4);}
.bg-home{background:linear-gradient(135deg,#43e97b,#38f9d7);}
.bg-app{background:linear-gradient(135deg,#30cfd0,#330867);}
.bg-toy{background:linear-gradient(135deg,#f6d365,#fda085);}
.bg-book{background:linear-gradient(135deg,#a18cd1,#fbc2eb);}
.bg-groc{background:linear-gradient(135deg,#ff6a88,#ffb199);}
.bg-spt{background:linear-gradient(135deg,#84fab0,#8fd3f4);}
.bg-tv{background:linear-gradient(135deg,#0f0c29,#302b63);}
@media(max-width:900px){.hero-area{grid-template-columns:1fr;}.hero-left,.hero-right{display:none;}.chart-grid,.set-grid{grid-template-columns:1fr;}.pd-modal{flex-direction:column;}.pd-imgs{width:100%;height:200px;border-radius:8px 8px 0 0;}.banner-strip{grid-template-columns:1fr 1fr;}}
@media(max-width:600px){
  .hdr-top{flex-wrap:wrap;gap:8px;}
  .hdr-search{order:3;flex:1 0 100%;}
  .hdr-nav{gap:8px;font-size:12px;flex-wrap:wrap;row-gap:6px;}
  .hdr-nav .hdr-link{font-size:11px;}
  .hdr-nav .hdr-btn{padding:6px 14px;font-size:12px;}
  .cat-nav{top:88px;}
  .co-modal{width:96vw;}
  .login-modal{width:96vw;padding:20px;}
  .spin-modal{width:96vw;padding:16px;}
  .wish-modal,.orders-modal,.pf-modal{width:96vw;padding:16px;}
  .filter-bar{flex-wrap:wrap;}
}
/* FILTERS & SORTING */
.filter-bar{display:flex;align-items:center;gap:10px;flex-wrap:wrap;padding:12px 16px;background:var(--white);border:1px solid var(--border);border-radius:var(--r);margin-top:10px;}
.filter-bar label{font-size:11px;color:var(--muted);font-weight:700;text-transform:uppercase;letter-spacing:.4px;margin-right:4px;}
.filter-group{display:flex;align-items:center;gap:6px;}
.filter-select{padding:7px 10px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:6px;font-size:12px;font-weight:600;}
.filter-price{width:70px;padding:7px 8px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:6px;font-size:12px;}
.filter-chip{padding:6px 12px;border:1px solid var(--border);background:var(--card);color:var(--muted);border-radius:20px;font-size:11px;font-weight:700;cursor:pointer;transition:.15s;}
.filter-chip.act{background:var(--blue);color:#fff;border-color:var(--blue);}
.filter-clear{margin-left:auto;font-size:12px;color:var(--orange);font-weight:700;cursor:pointer;}
/* REVIEWS */
.review-summary{display:flex;gap:30px;align-items:center;margin:16px 0;flex-wrap:wrap;}
.review-big-num{font-size:42px;font-weight:800;color:var(--text);line-height:1;}
.review-stars-row{display:flex;gap:2px;color:var(--gold);font-size:14px;margin:4px 0;}
.review-count-txt{font-size:12px;color:var(--muted);}
.review-bars{flex:1;min-width:160px;}
.review-bar-row{display:flex;align-items:center;gap:8px;font-size:12px;margin-bottom:5px;color:var(--muted);}
.review-bar-track{flex:1;height:6px;background:var(--card);border-radius:3px;overflow:hidden;}
.review-bar-fill{height:100%;background:var(--green);}
.review-item{border-bottom:1px solid var(--border);padding:14px 0;}
.review-item-head{display:flex;align-items:center;gap:10px;margin-bottom:6px;}
.review-avatar{width:30px;height:30px;border-radius:50%;background:linear-gradient(135deg,var(--blue),var(--orange));color:#fff;font-weight:800;font-size:12px;display:flex;align-items:center;justify-content:center;}
.review-item-name{font-weight:700;font-size:13px;color:var(--text);}
.review-item-date{font-size:11px;color:var(--muted);}
.review-item-stars{color:var(--gold);font-size:12px;margin-bottom:6px;}
.review-item-text{font-size:13px;color:var(--text);line-height:1.6;}
.review-form{background:var(--card);border-radius:8px;padding:16px;margin-bottom:18px;}
.review-form textarea{width:100%;padding:10px 12px;border:1px solid var(--border);background:var(--white);color:var(--text);border-radius:6px;font-size:13px;font-family:inherit;resize:vertical;min-height:70px;margin:10px 0;}
.review-star-picker{display:flex;gap:4px;font-size:24px;cursor:pointer;}
.review-star-picker span{color:var(--border);transition:.1s;}
.empty-cat-msg{padding:50px 20px;text-align:center;color:var(--muted);font-size:13px;width:100%;}
.review-star-picker span.sel{color:var(--gold);}
/* BECOME A SELLER PAGE */
.seller-page{position:fixed;inset:0;background:var(--bg);z-index:550;display:none;overflow-y:auto;}
.seller-page.on{display:block;}
.seller-top{background:#08090b;border-bottom:1px solid var(--border);padding:0 24px;height:58px;display:flex;align-items:center;gap:16px;position:sticky;top:0;z-index:5;}
.seller-top .logo{font-size:18px;}
.seller-close{margin-left:auto;background:var(--card);color:var(--text);padding:8px 16px;border-radius:6px;font-size:13px;font-weight:700;}
.seller-hero{background:linear-gradient(135deg,#1a1240,#0c0d10 60%);padding:60px 24px;text-align:center;border-bottom:1px solid var(--border);}
.seller-hero h1{font-family:'Poppins',sans-serif;font-size:38px;font-weight:800;color:#fff;margin-bottom:12px;}
.seller-hero h1 span{background:linear-gradient(135deg,var(--blue),#ff76ea);-webkit-background-clip:text;background-clip:text;-webkit-text-fill-color:transparent;}
.seller-hero p{font-size:15px;color:#b8b9c4;max-width:560px;margin:0 auto 26px;line-height:1.7;}
.seller-cta-row{display:flex;gap:14px;justify-content:center;flex-wrap:wrap;}
.seller-cta-primary{background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;font-weight:800;padding:14px 32px;border-radius:8px;font-size:15px;}
.seller-cta-secondary{background:transparent;border:1px solid var(--border);color:#fff;font-weight:700;padding:14px 32px;border-radius:8px;font-size:15px;}
.seller-stats{max-width:1100px;margin:0 auto;padding:36px 24px;display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:18px;}
.seller-stat{text-align:center;}
.seller-stat .num{font-size:30px;font-weight:800;color:var(--blue);}
.seller-stat .lbl{font-size:12px;color:var(--muted);margin-top:4px;}
.seller-benefits{max-width:1100px;margin:0 auto;padding:20px 24px 50px;}
.seller-benefits h2{font-family:'Poppins',sans-serif;font-size:24px;font-weight:800;color:var(--text);text-align:center;margin-bottom:30px;}
.seller-benefit-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:16px;}
.seller-benefit-card{background:var(--white);border:1px solid var(--border);border-radius:10px;padding:22px;}
.seller-benefit-card .icon{font-size:30px;margin-bottom:10px;}
.seller-benefit-card h4{font-size:14px;font-weight:700;color:var(--text);margin-bottom:6px;}
.seller-benefit-card p{font-size:12px;color:var(--muted);line-height:1.6;}
.seller-steps{max-width:1100px;margin:0 auto;padding:20px 24px 50px;}
.seller-steps h2{font-family:'Poppins',sans-serif;font-size:24px;font-weight:800;color:var(--text);text-align:center;margin-bottom:30px;}
.seller-step-row{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:16px;}
.seller-step{text-align:center;}
.seller-step-num{width:44px;height:44px;border-radius:50%;background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;font-weight:800;font-size:18px;display:flex;align-items:center;justify-content:center;margin:0 auto 12px;}
.seller-step h4{font-size:13px;font-weight:700;color:var(--text);margin-bottom:4px;}
.seller-step p{font-size:11px;color:var(--muted);}
.seller-form-wrap{max-width:680px;margin:0 auto;padding:20px 24px 70px;}
.seller-form-card{background:var(--white);border:1px solid var(--border);border-radius:12px;padding:32px;}
.seller-form-card h2{font-family:'Poppins',sans-serif;font-size:22px;font-weight:800;color:var(--text);margin-bottom:6px;}
.seller-form-card .sfsub{font-size:13px;color:var(--muted);margin-bottom:24px;}
.sf-row{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-bottom:14px;}
.sf-full{grid-column:1/-1;}
.sf-label{font-size:11px;font-weight:700;color:var(--muted);display:block;margin-bottom:5px;text-transform:uppercase;letter-spacing:.4px;}
.sf-input{width:100%;padding:11px 13px;border:1px solid var(--border);background:var(--card);color:var(--text);border-radius:6px;font-size:13px;}
.sf-input:focus{border-color:var(--blue);}
.sf-submit{width:100%;background:linear-gradient(135deg,var(--blue),var(--blue-d));color:#fff;font-weight:800;padding:14px;border-radius:8px;font-size:14px;text-transform:uppercase;margin-top:8px;}
.sf-success{text-align:center;padding:30px 0;}
.sf-success .ic{font-size:54px;margin-bottom:14px;}
/* SELLER REQUEST CARDS (admin) */
.sreq-card{border-bottom:1px solid var(--border);padding:16px;display:flex;gap:14px;align-items:flex-start;}
.sreq-card:last-child{border-bottom:none;}
.sreq-idimg{width:80px;height:54px;object-fit:cover;border-radius:6px;border:1px solid var(--border);flex-shrink:0;cursor:pointer;}
.sreq-info{flex:1;min-width:0;}
.sreq-name{font-weight:700;font-size:14px;color:var(--text);}
.sreq-biz{font-size:12px;color:var(--blue);font-weight:600;}
.sreq-meta{font-size:11px;color:var(--muted);margin-top:4px;line-height:1.6;}
.sreq-actions{display:flex;gap:8px;flex-shrink:0;align-items:center;}
.sreq-btn{padding:7px 16px;border-radius:5px;font-size:12px;font-weight:700;}
.sreq-accept{background:var(--green);color:#06281c;}
.sreq-reject{background:var(--red);color:#fff;}
</style>
<script>
/* ─── FIREBASE INIT ────────────────────────────────────────────────────── */
const firebaseConfig = {
  apiKey: "AIzaSyDsAc5vD41hb3ikubmFENRpLcvbn9OGrCQ",
  authDomain: "trendra-1890a.firebaseapp.com",
  projectId: "trendra-1890a",
  storageBucket: "trendra-1890a.firebasestorage.app",
  messagingSenderId: "678660854722",
  appId: "1:678660854722:web:652af74c5fce21b0a73c5b",
  measurementId: "G-DRG5YH7DNR"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
const PRODUCTS_COL = db.collection('products');
const SELLER_COL   = db.collection('sellerApplications');

let firebaseReady=false;
let dbUnsubscribe=null;

/* ===== SEED DATA (only written to Firestore if the collection is empty) ===== */
const seedProducts=[
  {id:1,name:"Galaxy Pulse X12 (128GB, 6GB RAM)",category:"mobiles",price:18999,mrp:24999,rating:4.3,reviews:12453,emoji:"📱",stock:45,desc:"6.5-inch FHD+ AMOLED, 50MP triple camera, 5000mAh, 33W fast charging.",brand:"Samsung",emi:"₹950/month"},
  {id:2,name:"NovaPhone Air 5G (256GB)",category:"mobiles",price:22499,mrp:29999,rating:4.4,reviews:8765,emoji:"📱",stock:30,desc:"120Hz AMOLED, 64MP quad camera, 4800mAh, ultra-slim 5G smartphone.",brand:"Nova",emi:"₹1,125/month"},
  {id:3,name:"TitanEdge Pro Max (512GB)",category:"mobiles",price:45999,mrp:59999,rating:4.5,reviews:5421,emoji:"📱",stock:18,desc:"Titanium frame, Pro camera system, 120Hz ProMotion display, 5G.",brand:"TitanTech",emi:"₹2,300/month"},
  {id:4,name:"ZenFlip Lite (64GB, Coral Pink)",category:"mobiles",price:9499,mrp:12999,rating:4.1,reviews:15234,emoji:"📱",stock:60,desc:"Compact, dual cameras, reliable 5000mAh battery, clean UI.",brand:"Zen",emi:"₹475/month"},
  {id:5,name:"SpeedX Turbo 5G (128GB)",category:"mobiles",price:13999,mrp:18999,rating:4.2,reviews:9870,emoji:"📱",stock:40,desc:"120Hz display, 48MP camera, 6000mAh mega battery, 5G-ready.",brand:"SpeedX",emi:"₹700/month"},
  {id:6,name:'CrystalView 55" 4K OLED TV',category:"tvs",price:54990,mrp:79990,rating:4.6,reviews:3421,emoji:"📺",stock:15,desc:"55-inch OLED 4K, Dolby Vision, Atmos sound, smart OS with all apps.",brand:"CrystalView",emi:"₹2,750/month"},
  {id:7,name:'BrightLife 43" Full HD Smart TV',category:"tvs",price:19990,mrp:28990,rating:4.3,reviews:8765,emoji:"📺",stock:28,desc:"43-inch FHD, Android TV, Netflix/Prime built-in, 20W speakers.",brand:"BrightLife",emi:"₹1,000/month"},
  {id:8,name:'ProVision 65" 4K QLED TV',category:"tvs",price:74990,mrp:99990,rating:4.7,reviews:1234,emoji:"📺",stock:10,desc:"65-inch QLED 4K, Quantum HDR, 120Hz, smart gaming features.",brand:"ProVision",emi:"₹3,750/month"},
  {id:9,name:"AeroBook Slim 14 (i5, 16GB, 512GB SSD)",category:"electronics",price:52990,mrp:68990,rating:4.4,reviews:3210,emoji:"💻",stock:22,desc:"14-inch FHD IPS, Intel i5, 16GB RAM, 512GB SSD, all-day battery.",brand:"AeroBook",emi:"₹2,650/month"},
  {id:10,name:'PixelView 27" FHD Monitor 165Hz',category:"electronics",price:16499,mrp:21999,rating:4.2,reviews:2987,emoji:"🖥️",stock:14,desc:"27-inch IPS, 165Hz refresh, 1ms response, HDR400, thin bezels.",brand:"PixelView",emi:"₹825/month"},
  {id:11,name:"SonicBeat Pro ANC Earbuds",category:"electronics",price:1799,mrp:3999,rating:4.0,reviews:21034,emoji:"🎧",stock:120,desc:"Hybrid ANC, 30hr battery, AAC/SBC, IPX5 waterproof.",brand:"SonicBeat",emi:"N/A"},
  {id:12,name:"CloudCast Smart Speaker 360°",category:"electronics",price:2499,mrp:3999,rating:4.3,reviews:6789,emoji:"🔊",stock:55,desc:"360° surround sound, voice assistant, multi-room pairing.",brand:"CloudCast",emi:"N/A"},
  {id:13,name:"AeroBook Gaming RTX (i7, 32GB, 1TB)",category:"electronics",price:89990,mrp:119990,rating:4.5,reviews:1876,emoji:"💻",stock:8,desc:"RTX 4070, Intel i7-13th Gen, 32GB DDR5, 1TB NVMe, 165Hz display.",brand:"AeroBook",emi:"₹4,500/month"},
  {id:14,name:"Men's Classic Denim Jacket (Blue)",category:"fashion",price:1299,mrp:2599,rating:4.1,reviews:4521,emoji:"🧥",stock:80,desc:"100% cotton denim, classic fit, double-stitched for durability.",brand:"StyleCo",emi:"N/A"},
  {id:15,name:"Women's Floral Maxi Dress",category:"fashion",price:999,mrp:2199,rating:4.3,reviews:7654,emoji:"👗",stock:65,desc:"Lightweight polyester blend, all-season wear, relaxed silhouette.",brand:"FloraCo",emi:"N/A"},
  {id:16,name:"Urban Runner Sneakers (Unisex)",category:"fashion",price:1599,mrp:3499,rating:4.2,reviews:9876,emoji:"👟",stock:90,desc:"EVA cushioned sole, breathable mesh, flexible upper, runs true to size.",brand:"UrbanStep",emi:"N/A"},
  {id:17,name:"Aviator UV400 Sunglasses",category:"fashion",price:499,mrp:1499,rating:3.9,reviews:3214,emoji:"🕶️",stock:200,desc:"Metal alloy frame, polarized UV400 lenses, unisex design.",brand:"ShadeCo",emi:"N/A"},
  {id:18,name:"Men's Slim Fit Formal Shirt (Pack of 3)",category:"fashion",price:1199,mrp:2499,rating:4.2,reviews:5678,emoji:"👔",stock:100,desc:"100% cotton, wrinkle-resistant, machine washable, premium finish.",brand:"FormalPlus",emi:"N/A"},
  {id:19,name:"FrostKing 260L Double Door Fridge",category:"appliances",price:24990,mrp:32990,rating:4.2,reviews:2341,emoji:"🧊",stock:10,desc:"Frost-free, Inverter compressor, 5-star energy rating, 3yr warranty.",brand:"FrostKing",emi:"₹1,250/month"},
  {id:20,name:"SpinMax 7kg Front Load Washing Machine",category:"appliances",price:14490,mrp:19990,rating:4.0,reviews:1876,emoji:"🌀",stock:12,desc:"1200 RPM, 15 wash programs, auto-clean drum, 5yr motor warranty.",brand:"SpinMax",emi:"₹725/month"},
  {id:21,name:"ArcticBreeze 1.5T 5-Star Inverter AC",category:"appliances",price:34990,mrp:44990,rating:4.4,reviews:3210,emoji:"❄️",stock:8,desc:"Wi-Fi enabled, PM 2.5 filter, 5-star BEE, dual inverter tech.",brand:"ArcticBreeze",emi:"₹1,750/month"},
  {id:22,name:"QuickBoil 1.8L Electric Kettle",category:"appliances",price:699,mrp:1199,rating:4.4,reviews:8901,emoji:"♨️",stock:95,desc:"1500W, auto shut-off, boil-dry protection, cordless 360° base.",brand:"QuickBoil",emi:"N/A"},
  {id:23,name:"AeroPure 3-in-1 Air Purifier",category:"appliances",price:8990,mrp:13990,rating:4.3,reviews:2134,emoji:"🌬️",stock:20,desc:"HEPA + Carbon + UV filter, 400 sq.ft coverage, sleep mode, AQI display.",brand:"AeroPure",emi:"₹450/month"},
  {id:24,name:"Wooden Bookshelf 5-Tier (Walnut)",category:"furniture",price:3499,mrp:5999,rating:4.4,reviews:1543,emoji:"📚",stock:20,desc:"Engineered wood, walnut finish, easy assembly, holds 200+ books.",brand:"WoodArt",emi:"N/A"},
  {id:25,name:"Memory Foam Sofa (3-Seater, Grey)",category:"furniture",price:19990,mrp:34990,rating:4.5,reviews:987,emoji:"🛋️",stock:6,desc:"High-density memory foam, premium fabric, no-sag springs, 5yr warranty.",brand:"ComfortPlus",emi:"₹1,000/month"},
  {id:26,name:"Study Table with Drawer & Shelf",category:"furniture",price:4499,mrp:7499,rating:4.2,reviews:2345,emoji:"🪑",stock:15,desc:"Engineered wood, cable management hole, 80kg load capacity.",brand:"StudyPro",emi:"N/A"},
  {id:27,name:"Non-Stick Cookware Set (7 Pieces)",category:"furniture",price:1899,mrp:3499,rating:4.3,reviews:4123,emoji:"🍳",stock:40,desc:"Hard-anodized body, PFOA-free coating, works on gas & induction.",brand:"KitchenPro",emi:"N/A"},
  {id:28,name:"Building Blocks Set 200 Pieces",category:"toys",price:899,mrp:1599,rating:4.6,reviews:3421,emoji:"🧩",stock:70,desc:"BPA-free plastic, ASTM certified, compatible with major brands.",brand:"BlockWorld",emi:"N/A"},
  {id:29,name:"RC Stunt Car with Remote",category:"toys",price:1299,mrp:2499,rating:4.3,reviews:2109,emoji:"🚗",stock:45,desc:"2.4GHz remote, 360° spins, 30 km/h, rechargeable battery.",brand:"SpeedKid",emi:"N/A"},
  {id:30,name:"Plush Teddy Bear XL (24 inch)",category:"toys",price:599,mrp:999,rating:4.7,reviews:5432,emoji:"🧸",stock:55,desc:"Super-soft, PP cotton fill, machine washable, CE certified.",brand:"CuddleBear",emi:"N/A"},
  {id:31,name:"Atomic Habits (Paperback)",category:"books",price:349,mrp:599,rating:4.7,reviews:21345,emoji:"📘",stock:160,desc:"James Clear's #1 NYT bestseller on building habits that stick.",brand:"Penguin",emi:"N/A"},
  {id:32,name:"The Midnight Library (Paperback)",category:"books",price:249,mrp:399,rating:4.5,reviews:9821,emoji:"📖",stock:130,desc:"Matt Haig's thought-provoking novel about life choices.",brand:"Canongate",emi:"N/A"},
  {id:33,name:"Premium Basmati Rice 5kg",category:"grocery",price:549,mrp:699,rating:4.3,reviews:6543,emoji:"🍚",stock:200,desc:"Extra-long grain, aged 1 year, FSSAI certified, rich aroma.",brand:"AromaStar",emi:"N/A"},
  {id:34,name:"Cold Pressed Olive Oil 1L",category:"grocery",price:699,mrp:999,rating:4.4,reviews:3210,emoji:"🫒",stock:85,desc:"Extra virgin, first cold press, USDA organic certified.",brand:"OliveGold",emi:"N/A"},
  {id:35,name:"Yoga Mat with Carry Strap (6mm)",category:"grocery",price:599,mrp:999,rating:4.5,reviews:4567,emoji:"🧘",stock:90,desc:"NBR foam, anti-slip texture, sweat-resistant, 6mm cushion.",brand:"FitLife",emi:"N/A"},
  {id:36,name:"Cricket Kit Bag Combo (Senior)",category:"grocery",price:2999,mrp:4999,rating:4.2,reviews:1234,emoji:"🏏",stock:22,desc:"Bat, ball, gloves, pads, helmet, holdall bag — all included.",brand:"SportPro",emi:"N/A"},
  {id:37,name:"Men's Black Slim Fit Trousers (Formal Pant)",category:"fashion",price:899,mrp:1799,rating:4.3,reviews:642,emoji:"👖",stock:75,desc:"Black formal trouser, slim fit, wrinkle-resistant, office wear.",brand:"FormalPlus",emi:"N/A"},
  {id:38,name:"Men's Black Cotton T-Shirt (Round Neck)",category:"fashion",price:399,mrp:799,rating:4.4,reviews:1203,emoji:"👕",stock:150,desc:"100% cotton, black round-neck tee, everyday casual wear.",brand:"StyleCo",emi:"N/A"},
  {id:39,name:"Women's Black A-Line Dress",category:"fashion",price:1099,mrp:2299,rating:4.5,reviews:876,emoji:"👗",stock:55,desc:"Black A-line dress, lightweight fabric, party & casual wear.",brand:"FloraCo",emi:"N/A"},
  {id:45,name:"Women's Red Wrap Dress",category:"fashion",price:1199,mrp:2499,rating:4.4,reviews:523,emoji:"👗",stock:45,desc:"Red wrap dress, flattering fit, party & evening wear.",brand:"FloraCo",emi:"N/A"},
  {id:40,name:"Men's Red Casual Shirt (Checked)",category:"fashion",price:799,mrp:1599,rating:4.1,reviews:534,emoji:"👔",stock:60,desc:"Red checked casual shirt, breathable cotton blend.",brand:"FormalPlus",emi:"N/A"},
  {id:41,name:"Women's Blue Denim Jeans (Skinny Fit)",category:"fashion",price:1199,mrp:2399,rating:4.3,reviews:945,emoji:"👖",stock:70,desc:"Blue skinny-fit denim jeans, stretchable, everyday wear.",brand:"UrbanStep",emi:"N/A"},
  {id:42,name:"Men's White Formal Shirt (Slim Fit)",category:"fashion",price:899,mrp:1799,rating:4.4,reviews:712,emoji:"👔",stock:80,desc:"White slim-fit formal shirt, easy-iron cotton blend.",brand:"FormalPlus",emi:"N/A"},
  {id:43,name:"Men's Black Bomber Jacket",category:"fashion",price:1899,mrp:3499,rating:4.5,reviews:389,emoji:"🧥",stock:35,desc:"Black bomber jacket, water-resistant shell, ribbed cuffs.",brand:"StyleCo",emi:"N/A"},
  {id:44,name:"Women's Green Kurti (Cotton)",category:"fashion",price:649,mrp:1299,rating:4.2,reviews:601,emoji:"👚",stock:90,desc:"Green cotton kurti, breathable, daily & festive wear.",brand:"FloraCo",emi:"N/A"}
];

/* products is now populated live from Firestore — starts empty until the snapshot listener fires */
let products=[];

async function initFirestoreProducts(){
  try{
    // Use a separate marker doc to track whether we've EVER seeded this project —
    // checking "is the products collection empty?" was wrong: if an admin deletes
    // all products on purpose, the collection becomes empty and would get
    // re-seeded with the original 44 starter products on the next page load/listener
    // restart. The marker makes seeding a true one-time action.
    const seedMarkerRef=db.collection('meta').doc('seedStatus');
    const markerSnap=await seedMarkerRef.get();
    if(!markerSnap.exists){
      const snap=await PRODUCTS_COL.limit(1).get();
      if(snap.empty){
        // first run ever for this project — seed Firestore with starter catalogue
        const batch=db.batch();
        seedProducts.forEach(p=>{
          const ref=PRODUCTS_COL.doc(String(p.id));
          batch.set(ref,p);
        });
        batch.set(seedMarkerRef,{seededAt:new Date().toISOString()});
        await batch.commit();
      }else{
        // products already exist from before this marker system existed — just record it
        await seedMarkerRef.set({seededAt:new Date().toISOString(),note:'backfilled, products pre-existed'});
      }
    }
  }catch(err){
    console.error('Firestore seed error:',err);
    toast('⚠️ Could not connect to database — using offline mode');
    products=[...seedProducts];
    renderSections();
  }
  // live listener: any add/edit/delete (from this tab, another tab, or another admin) reflects instantly
  dbUnsubscribe=PRODUCTS_COL.onSnapshot(snapshot=>{
    products=snapshot.docs.map(d=>({...d.data(),id:Number(d.id)||d.id}));
    firebaseReady=true;
    renderSections();
    populateBrandFilter();
    if(document.getElementById('admPanel')?.classList.contains('on')){
      renderPrdTbl();renderInvTbl();renderAdmDashboard();
    }
    const pdpSec=document.getElementById('pdpSec');
    if(pdpSec&&pdpSec.style.display!=='none'&&currentPDPId){
      renderProductDetailPage(currentPDPId);
    }
  },err=>{
    console.error('Firestore listener error:',err);
    toast('⚠️ Database connection lost — check your Firestore rules');
  });
}
</script>
</head>
<body>

<div class="offer-strip">
  <span>🚀 Free delivery on orders above ₹500</span>
  <span>⚡ New user offer: Extra 10% off</span>
  <span>📧 Contact us: trendra.care.ac.in@gmail.com</span>
  <span>🎁 Spin & Win instant coupons</span>
</div>

<header class="hdr">
  <div class="hdr-top">
    <div class="logo" onclick="goHome()">TRENDRA <sub>SHOPKART</sub></div>
    <div class="hdr-search">
      <input type="text" id="searchQ" placeholder="Search for products, brands and more">
      <button onclick="doSearch()">🔍</button>
    </div>
    <div class="hdr-nav">
      <div id="hdrUserArea">
        <button class="hdr-btn" onclick="openM('loginModal')">Login</button>
      </div>
      <div class="hdr-link" onclick="openSellerPage()" style="background:rgba(255,255,255,.1);padding:6px 14px;border-radius:20px;">🏪 Become a Seller</div>
      <div class="hdr-link" onclick="openM('baristaModal');initBarista()" style="background:rgba(255,255,255,.1);padding:6px 14px;border-radius:20px;">🤖 Trendra AI</div>
      <div class="hdr-link" onclick="openM('settingsModal')">⚙️</div>
      <div class="hdr-link" onclick="toggleNotif()">🔔<span class="badge" id="notifBadge">3</span></div>
      <div class="hdr-link" onclick="openM('wishModal');renderWishModal()">❤️<span class="badge" id="wishBadge">0</span></div>
      <div class="hdr-link" onclick="openCart()">🛒 Cart<span class="badge" id="cartBadge">0</span></div>
    </div>
  </div>
</header>

<div class="notif-panel" id="notifPanel">
  <div class="notif-hdr"><span>Notifications</span><button onclick="closeNotif()" style="background:none;font-size:18px;">✕</button></div>
  <div class="notif-item unread"><div class="notif-dot"></div><div><b>Order Shipped!</b><br>Your order #TS2024001 is on the way.</div></div>
  <div class="notif-item unread"><div class="notif-dot"></div><div><b>Coupon Earned!</b><br>You won SPIN20 – 20% off coupon.</div></div>
  <div class="notif-item unread"><div class="notif-dot"></div><div><b>Flash Sale!</b><br>Electronics up to 60% off. Hurry!</div></div>
  <div class="notif-item"><div class="notif-dot" style="background:#ccc;"></div><div><b>Delivery Done</b><br>Your order was delivered successfully.</div></div>
</div>

<div class="profile-drop" id="profileDrop">
  <div class="pd-user">
    <div class="hdr-avatar" id="pdAvatar">A</div>
    <div><b id="pdName">User</b><br><span style="font-size:11px;color:var(--muted);" id="pdEmail">user@email.com</span></div>
  </div>
  <a onclick="openM('ordersModal');renderOrdersModal()">📦 My Orders</a>
  <a onclick="openM('wishModal');renderWishModal()">❤️ Wishlist</a>
  <a onclick="openM('loginModal')">⚙️ Account Settings</a>
  <a id="adminPanelLink" style="display:none;" onclick="openAdmin()">🔧 Admin Panel</a>
  <a onclick="doLogout()" style="color:var(--red);">🚪 Logout</a>
</div>

<nav class="cat-nav">
  <div class="cat-nav-inner">
    <div class="cat-item" onclick="scrollSec('deals')"><span class="ci">🔥</span>Today's Deals</div>
    <div class="cat-item" onclick="scrollSec('mobiles')"><span class="ci">📱</span>Mobiles</div>
    <div class="cat-item" onclick="scrollSec('tvs')"><span class="ci">📺</span>TVs</div>
    <div class="cat-item" onclick="scrollSec('electronics')"><span class="ci">💻</span>Electronics</div>
    <div class="cat-item" onclick="scrollSec('fashion')"><span class="ci">👕</span>Fashion</div>
    <div class="cat-item" onclick="scrollSec('appliances')"><span class="ci">🔌</span>Appliances</div>
    <div class="cat-item" onclick="scrollSec('furniture')"><span class="ci">🛋️</span>Furniture</div>
    <div class="cat-item" onclick="scrollSec('toysbooks')"><span class="ci">🧸</span>Toys & Books</div>
    <div class="cat-item" onclick="scrollSec('grocery')"><span class="ci">🛒</span>Grocery</div>
    <div class="cat-item" onclick="openM('baristaModal');initBarista()"><span class="ci">🤖</span>Trendra AI</div>
    <div class="cat-item" onclick="openM('spinModal')"><span class="ci">🎯</span>Spin & Win</div>
  </div>
</nav>

<main>
<div class="wrap" style="margin-top:10px;">
  <div class="hero-area">
    <div class="hero-left">
      <div class="side-cat">📱 Mobiles</div><div class="side-cat">💻 Laptops</div>
      <div class="side-cat">📺 TVs & ACs</div><div class="side-cat">👗 Fashion</div>
      <div class="side-cat">🏠 Home Decor</div><div class="side-cat">🍳 Kitchen</div>
      <div class="side-cat">🧸 Toys</div><div class="side-cat">📚 Books</div>
      <div class="side-cat">💄 Beauty</div><div class="side-cat">🏋️ Sports</div>
    </div>
    <div class="hero-banner" id="heroBanner">
      <div class="hero-slide active" style="background:linear-gradient(135deg,#2874f0,#0d47a1);"><div><h1>Mega Sale Days</h1><p>Up to 70% off across all categories</p><span class="hbtn">Shop Now →</span></div></div>
      <div class="hero-slide" style="background:linear-gradient(135deg,#e91e63,#880e4f);"><div><h1>Fashion Fiesta</h1><p>Trending styles for men & women from ₹399</p><span class="hbtn">Explore Now →</span></div></div>
      <div class="hero-slide" style="background:linear-gradient(135deg,#ff6f00,#e65100);"><div><h1>Appliance Fest</h1><p>Best deals on TVs, ACs, Washing Machines</p><span class="hbtn">View Offers →</span></div></div>
      <div class="hero-slide" style="background:linear-gradient(135deg,#1b5e20,#388e3c);"><div><h1>Electronics Bonanza</h1><p>Laptops, Earbuds, Cameras & Smart Devices</p><span class="hbtn">Grab Deals →</span></div></div>
      <div class="hero-slide" style="background:linear-gradient(135deg,#4a148c,#7b1fa2);"><div><h1>Spin & Win!</h1><p>Get instant coupons — one free spin per visit</p><span class="hbtn" onclick="openM('spinModal')">Play Now 🎯</span></div></div>
      <div class="hero-dots" id="heroDots"></div>
      <div class="hero-arr prev" onclick="heroSlide(-1)">&#8249;</div>
      <div class="hero-arr next" onclick="heroSlide(1)">&#8250;</div>
    </div>
    <div class="hero-right">
      <div class="side-cat">🎁 Gift Cards</div><div class="side-cat">💊 Healthcare</div>
      <div class="side-cat">🚗 Automotive</div><div class="side-cat">🐾 Pet Supplies</div>
      <div class="side-cat">📷 Cameras</div><div class="side-cat">🎮 Gaming</div>
      <div class="side-cat">🌿 Organic</div><div class="side-cat">🎨 Art & Craft</div>
      <div class="side-cat">🧴 Personal Care</div><div class="side-cat">💼 Luggage</div>
    </div>
  </div>

  <div class="filter-bar" id="filterBar">
    <label>Sort</label>
    <select class="filter-select" id="sortSelect" onchange="applyFilters()">
      <option value="relevance">Relevance</option>
      <option value="price_low">Price: Low to High</option>
      <option value="price_high">Price: High to Low</option>
      <option value="rating">Rating: High to Low</option>
      <option value="newest">Newest First</option>
    </select>
    <label>Price</label>
    <div class="filter-group">
      <input type="number" class="filter-price" id="priceMin" placeholder="Min" onchange="applyFilters()">
      <span style="color:var(--muted);">–</span>
      <input type="number" class="filter-price" id="priceMax" placeholder="Max" onchange="applyFilters()">
    </div>
    <label>Brand</label>
    <select class="filter-select" id="brandSelect" onchange="applyFilters()">
      <option value="">All Brands</option>
    </select>
    <div class="filter-group">
      <span class="filter-chip" data-rating="4" onclick="toggleRatingChip(this)">4★ & up</span>
      <span class="filter-chip" data-rating="3" onclick="toggleRatingChip(this)">3★ & up</span>
    </div>
    <span class="filter-clear" onclick="clearFilters()">Clear Filters ✕</span>
  </div>

  <div id="searchSec" style="display:none;">
    <div class="sec" style="padding:16px;">
      <div class="sec-hdr"><div><h2>Search Results</h2><div class="sub" id="searchSub"></div></div><button class="sec-hdr view-all" onclick="clearSearch()">Clear ✕</button></div>
      <div class="prod-grid" id="srGrid"></div>
    </div>
  </div>

  <div id="pdpSec" style="display:none;"></div>

  <div id="homeSec">
    <div class="sec" id="deals">
      <div class="sec-hdr">
        <div><h2>⚡ Deals of the Day</h2><div class="sub">Limited time offers — grab fast!</div></div>
        <div style="display:flex;align-items:center;gap:12px;">
          <div class="sec-hdr timer">Ends in <b id="countdownTime">--:--:--</b></div>
          <button class="sec-hdr view-all">View All</button>
        </div>
      </div>
      <div class="deal-strip" id="dealStrip"></div>
    </div>
    <div class="sec" id="mobiles" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>📱 Top Mobiles</h2><div class="sub">Latest smartphones from trusted brands</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-mobiles"></div>
    </div>
    <div class="banner-strip">
      <div class="mini-banner" style="background:linear-gradient(135deg,#2874f0,#1a237e);" onclick="scrollSec('electronics')"><h3>Best Laptops</h3><p>Work & play smarter</p><span>From ₹32,999</span></div>
      <div class="mini-banner" style="background:linear-gradient(135deg,#e91e63,#880e4f);" onclick="scrollSec('fashion')"><h3>Fashion Sale</h3><p>Up to 80% off</p><span>Shop Now</span></div>
      <div class="mini-banner" style="background:linear-gradient(135deg,#ff6f00,#bf360c);" onclick="scrollSec('appliances')"><h3>Home Appliances</h3><p>Great deals inside</p><span>Up to 50% off</span></div>
      <div class="mini-banner" style="background:linear-gradient(135deg,#00695c,#1b5e20);" onclick="openM('spinModal')"><h3>Spin & Win</h3><p>Instant coupons!</p><span>Play Free 🎯</span></div>
    </div>
    <div class="sec" id="tvs" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>📺 Smart TVs</h2><div class="sub">4K, OLED & more</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-tvs"></div>
    </div>
    <div class="sec" id="electronics" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>💻 Electronics</h2><div class="sub">Laptops, earbuds, smart devices</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-electronics"></div>
    </div>
    <div class="sec" id="fashion" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>👕 Fashion Trends</h2><div class="sub">Styles for men, women & kids</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-fashion"></div>
    </div>
    <div class="sec" id="appliances" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>🔌 Home Appliances</h2><div class="sub">ACs, fridges, washing machines & more</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-appliances"></div>
    </div>
    <div class="sec" id="furniture" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>🛋️ Furniture & Home</h2><div class="sub">Decor, kitchen & living</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-furniture"></div>
    </div>
    <div class="sec" id="toysbooks" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>🧸 Toys & Books</h2><div class="sub">For curious minds</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-toysbooks"></div>
    </div>
    <div class="sec" id="grocery" style="margin-top:10px;">
      <div class="sec-hdr"><div><h2>🛒 Grocery & Sports</h2><div class="sub">Daily needs & fitness gear</div></div><button class="sec-hdr view-all">View All</button></div>
      <div class="prod-scroll" id="s-grocery"></div>
    </div>
    <div class="feature-row" style="margin-top:10px;">
      <div class="fbox"><div class="fi">🚚</div><h4>Free Delivery</h4><p>On orders above ₹500. Fast & reliable.</p></div>
      <div class="fbox"><div class="fi">↩️</div><h4>Easy Returns</h4><p>7-day hassle-free return policy.</p></div>
      <div class="fbox"><div class="fi">🔒</div><h4>Secure Payments</h4><p>100% safe UPI, Card & Net Banking.</p></div>
      <div class="fbox"><div class="fi">🏆</div><h4>Top Brands</h4><p>Millions of authentic products.</p></div>
      <div class="fbox"><div class="fi">💬</div><h4>24/7 Support</h4><p>Always here to help you out.</p></div>
    </div>
  </div>
</div>

<div class="trust-bar">
  <div class="trust-inner">
    <div class="trust-item"><div class="trust-icon">📦</div><div class="trust-info"><b>5 Crore+ Products</b><span>Across all categories</span></div></div>
    <div class="trust-item"><div class="trust-icon">🚀</div><div class="trust-info"><b>Express Delivery</b><span>In select cities</span></div></div>
    <div class="trust-item"><div class="trust-icon">💳</div><div class="trust-info"><b>EMI Available</b><span>No-cost EMI on select items</span></div></div>
    <div class="trust-item"><div class="trust-icon">✅</div><div class="trust-info"><b>100% Authentic</b><span>Only genuine products</span></div></div>
    <div class="trust-item"><div class="trust-icon">🎁</div><div class="trust-info"><b>Gift Cards</b><span>For your loved ones</span></div></div>
  </div>
</div>

<footer class="footer">
  <div class="footer-top">
    <div class="footer-col"><h5>About</h5><a>About TRENDRA SHOPKART</a><a>Careers</a><a>Press</a><a>Blog</a><a onclick="openSellerPage()">Become a Seller</a><a>Corporate Info</a></div>
    <div class="footer-col"><h5>Help</h5><a>Payments</a><a>Shipping</a><a>Cancellation & Returns</a><a>FAQ</a><a>Report Infringement</a></div>
    <div class="footer-col"><h5>Policy</h5><a>Return Policy</a><a>Terms of Use</a><a>Privacy</a><a>Sitemap</a><a>Cookie Policy</a></div>
    <div class="footer-col"><h5>Social</h5><a>Facebook</a><a>Twitter / X</a><a>Instagram</a><a>YouTube</a></div>
    <div class="footer-col"><h5>Mail Us</h5><p style="font-size:12px;line-height:1.8;">TRENDRA SHOPKART Internet Pvt Ltd,<br>Buildings Alyssa, Prayagraj,<br>Uttar Pradesh - 211001</p></div>
    <div class="footer-col"><h5>Contact</h5><div style="background:rgba(255,255,255,.08);border-radius:5px;padding:12px;font-size:12px;"><div style="margin-bottom:6px;">Email:</div><div class="upi" style="font-size:13px;">trendra.care.ac.in@gmail.com</div><div style="margin-top:8px;color:#6b8a9a;">We reply within 24 hours</div></div></div>
  </div>
  <div class="footer-bottom">© 2026 TRENDRA SHOPKART Internet Pvt. Ltd. All rights reserved. | CIN: U51109KA2024PTC001001</div>
</footer>
</main>

<div class="overlay" id="overlay" onclick="closeAll()"></div>

<aside class="cart-side" id="cartSide">
  <div class="cart-hdr"><span>🛒 My Cart</span><button onclick="closeCart()">✕</button></div>
  <div class="cart-body" id="cartBody"></div>
  <div class="cart-foot" id="cartFoot"></div>
</aside>

<div class="modal login-modal" id="loginModal">
  <button class="mclose" onclick="closeM('loginModal')">✕</button>
  <h2 id="loginTitle">Login</h2>
  <p class="lsub">Get access to your orders, wishlist & recommendations</p>
  <div class="l-tabs"><button class="act" id="tLogin" onclick="lTab('login')">Login</button><button id="tSignup" onclick="lTab('signup')">Sign Up</button></div>

  <div id="loginFields">
    <div class="lfield"><label>Email Address</label><input type="email" id="lemail" placeholder="Enter your email"></div>
    <div class="lfield"><label>Password</label><input type="password" id="lpass" placeholder="Enter password"></div>
    <button class="l-submit" id="lBtn" onclick="doLogin()">LOGIN</button>
  </div>

  <div id="signupFields" style="display:none;">
    <div class="lfield"><label>Full Name</label><input type="text" id="sname" placeholder="Your full name"></div>
    <div class="lfield"><label>Email Address</label><input type="email" id="semail" placeholder="Enter your email"></div>
    <div class="lfield"><label>Password</label><input type="password" id="spass" placeholder="Create a password (min 6 chars)"></div>
    <div class="lfield">
      <label>Verify via</label>
      <div class="otp-choice">
        <button type="button" class="otp-choice-btn act" id="otpChoiceEmail" onclick="setOtpMethod('email')">📧 Email OTP</button>
        <button type="button" class="otp-choice-btn" id="otpChoiceMobile" onclick="setOtpMethod('mobile')" disabled title="Mobile OTP coming soon">📱 Mobile OTP <span class="otp-soon">Coming soon</span></button>
      </div>
    </div>
    <div class="lfield" id="smobileRow" style="display:none;"><label>Mobile Number</label><input type="tel" id="smobile" placeholder="10-digit mobile number"></div>
    <button class="l-submit" id="signupBtn" onclick="doSignupSendOtp()">SEND VERIFICATION LINK</button>
    <p class="otp-hint" id="otpHint">We'll email you a secure sign-in link — click it to verify and create your account. No password to remember at login next time, but you can still set one above for future use.</p>
  </div>

  <div id="otpSentPanel" style="display:none;text-align:center;padding:10px 0;">
    <div style="font-size:40px;margin-bottom:8px;">📬</div>
    <h3 style="margin-bottom:6px;">Check your email</h3>
    <p class="lsub" id="otpSentMsg">We've sent a verification link to your email.</p>
    <button class="social-btn" style="margin-top:14px;width:100%;" onclick="lTab('signup')">← Back</button>
  </div>

  <div class="l-divider" id="loginDivider">OR</div>
  <div class="social-btns" id="loginSocialBtns">
    <button class="social-btn" onclick="socialLogin('Google')">🔵 Google</button>
    <button class="social-btn" onclick="socialLogin('Facebook')">🔷 Facebook</button>
  </div>
</div>

<div class="modal pd-modal" id="pdModal">
  <button class="mclose" onclick="closeM('pdModal')" style="z-index:5;">✕</button>
  <div id="pdContent" style="display:contents;"></div>
</div>

<div class="modal spin-modal" id="spinModal">
  <button class="mclose" onclick="closeM('spinModal')">✕</button>
  <h2>🎉 Spin & Win</h2>
  <p class="ssub">One free spin — instant coupon for your cart!</p>
  <div class="wheel-wrap">
    <div class="wheel-ptr"></div>
    <div class="wheel" id="wheel"></div>
    <div class="wheel-center">🎯</div>
  </div>
  <button class="spin-btn" id="spinBtn" onclick="doSpin()">SPIN NOW</button>
  <div class="spin-res" id="spinRes"></div>
</div>

<div class="modal co-modal" id="coModal">
  <button class="mclose" onclick="closeM('coModal')">✕</button>
  <div class="co-steps">
    <div class="co-step act" id="cs1"><div class="co-step-num">1</div>Address</div>
    <div class="co-line" id="cl1"></div>
    <div class="co-step" id="cs2"><div class="co-step-num">2</div>Payment</div>
    <div class="co-line" id="cl2"></div>
    <div class="co-step" id="cs3"><div class="co-step-num">3</div>Confirm</div>
  </div>
  <div class="co-body" id="coBody"></div>
</div>

<div class="modal order-ok" id="orderOkModal">
  <div class="ok-icon">🎉</div>
  <h2>Order Placed Successfully!</h2>
  <p class="osub">Thank you for shopping with ShopKart!<br>Your order will be delivered in <b>2–5 business days</b>.</p>
  <div class="order-id" id="orderIdEl">ORDER #----</div>
  <p id="orderPayInfo" style="font-size:12px;color:var(--muted);margin-bottom:6px;"></p>
  <button class="cont-btn order-ok" onclick="closeAll();renderSections();">Continue Shopping 🛍️</button>
</div>

<div class="modal wish-modal" id="wishModal">
  <button class="mclose" onclick="closeM('wishModal')">✕</button>
  <h2>❤️ My Wishlist (<span id="wishCount">0</span>)</h2>
  <div class="wish-grid" id="wishGrid"></div>
</div>

<div class="modal orders-modal" id="ordersModal">
  <button class="mclose" onclick="closeM('ordersModal')">✕</button>
  <h2>📦 My Orders</h2>
  <div id="ordersContent"></div>
</div>

<div class="modal" id="settingsModal" style="width:420px;padding:26px;">
  <button class="mclose" onclick="closeM('settingsModal')">✕</button>
  <h2 style="font-size:19px;font-weight:800;margin-bottom:4px;">⚙️ Settings</h2>
  <p style="font-size:13px;color:var(--muted);margin-bottom:18px;">Customize your ShopKart experience</p>
  <div class="set-row" style="padding:12px 0;"><span>🌙 Dark Mode</span><label class="toggle-wrap"><input type="checkbox" id="darkModeToggle" onchange="toggleDarkMode()"><span class="tog-sl"></span></label></div>
  <div class="set-row" style="padding:12px 0;"><span>🔔 Push Notifications</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
  <div class="set-row" style="padding:12px 0;"><span>📧 Email Offers</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
  <div class="set-row" style="padding:12px 0;"><span>🔊 Sound Effects</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
  <div class="set-row" style="padding:12px 0;border-bottom:none;"><span>🌐 Language</span>
    <select class="sel-sm"><option>English</option><option>हिन्दी</option></select>
  </div>
  <div style="margin-top:16px;padding-top:16px;border-top:1px solid var(--border);font-size:12px;color:var(--muted);">
    Contact us: <b style="color:var(--blue);">trendra.care.ac.in@gmail.com</b>
  </div>
</div>

<div class="modal pf-modal" id="pfModal">
  <button class="mclose" onclick="closeM('pfModal')">✕</button>
  <h2 id="pfTitle">Add Product</h2>
  <div class="pf-grid">
    <div class="pf-full"><label class="pf-label">Product Name</label><input class="pf-input" id="pf_name" placeholder="Full product name"></div>
    <div><label class="pf-label">Category</label>
      <select class="pf-input" id="pf_cat">
        <option value="mobiles">Mobiles</option><option value="tvs">TVs</option><option value="electronics">Electronics</option><option value="fashion">Fashion</option><option value="appliances">Appliances</option><option value="furniture">Furniture</option><option value="toys">Toys</option><option value="books">Books</option><option value="grocery">Grocery</option><option value="sports">Sports</option>
      </select>
    </div>
    <div><label class="pf-label">Emoji</label><input class="pf-input" id="pf_emoji" maxlength="4"></div>
    <div class="pf-full"><label class="pf-label">Product Photo</label>
      <input type="file" accept="image/*" class="pf-input" id="pf_photo_file" onchange="handlePhotoUpload(event)" style="padding:6px;">
      <div id="pf_photo_preview" style="margin-top:8px;display:none;">
        <img id="pf_photo_img" style="width:90px;height:90px;object-fit:cover;border-radius:6px;border:1px solid var(--border);">
        <button type="button" onclick="removePhoto()" style="margin-left:8px;background:var(--bg);color:var(--red);padding:6px 12px;border-radius:3px;font-size:12px;font-weight:700;">Remove Photo</button>
      </div>
    </div>
    <div><label class="pf-label">Selling Price ₹</label><input class="pf-input" type="number" id="pf_price"></div>
    <div><label class="pf-label">MRP ₹</label><input class="pf-input" type="number" id="pf_mrp"></div>
    <div><label class="pf-label">Stock</label><input class="pf-input" type="number" id="pf_stock"></div>
    <div><label class="pf-label">Rating (0–5)</label><input class="pf-input" type="number" step="0.1" id="pf_rating"></div>
    <div><label class="pf-label">Reviews Count</label><input class="pf-input" type="number" id="pf_reviews"></div>
    <div class="pf-full"><label class="pf-label">Description</label><textarea class="pf-input" id="pf_desc" rows="3"></textarea></div>
  </div>
  <div class="pf-actions">
    <button class="pf-cancel" onclick="closeM('pfModal')">Cancel</button>
    <button class="pf-save" onclick="saveProd()">Save Product</button>
  </div>
</div>

<div class="modal barista-modal" id="baristaModal">
  <button class="mclose" onclick="closeM('baristaModal')">✕</button>
  <div class="bar-hdr">
    <div class="bar-avatar">🤖</div>
    <div><h2>Trendra AI</h2><p class="bar-sub">Product dhundo ya kuch bhi poocho — order, delivery, return sab</p></div>
  </div>
  <div class="bar-chat" id="barChat"></div>
  <div class="bar-input-row">
    <input type="text" class="bar-input" id="barInput" placeholder="e.g. kala pant, red dress, order status..." onkeydown="if(event.key==='Enter')sendBarista()">
    <button class="bar-send" onclick="sendBarista()">➤</button>
  </div>
</div>

<div class="toast" id="toast"></div>
<button class="btt" id="btt" onclick="window.scrollTo({top:0,behavior:'smooth'})">↑</button>
<button class="spin-fab" onclick="openM('spinModal')">🎁 Spin & Win</button>

<div class="adm" id="admPanel">
  <div class="adm-top">
    <div class="adm-brand">Shop<span>Kart</span> Admin</div>
    <span style="font-size:12px;color:#6b7280;margin-right:auto;">aksahuakhil@gmail.com</span>
    <button class="adm-top-btn" onclick="exitAdmin()">← Back to Store</button>
    <button class="adm-top-btn danger" onclick="logoutAdmin()">Logout</button>
  </div>
  <div class="adm-body">
    <aside class="adm-side">
      <div class="ns">Main</div>
      <button class="adm-nav on" onclick="admNav('dashboard',this)">📊 Dashboard</button>
      <button class="adm-nav" onclick="admNav('orders',this)">📦 Orders</button>
      <div class="ns">Catalogue</div>
      <button class="adm-nav" onclick="admNav('products',this)">🛍️ Products</button>
      <button class="adm-nav" onclick="admNav('inventory',this)">📋 Inventory</button>
      <div class="ns">Business</div>
      <button class="adm-nav" onclick="admNav('customers',this)">👥 Customers</button>
      <button class="adm-nav" onclick="admNav('sellerreq',this)">🏪 Seller Requests</button>
      <button class="adm-nav" onclick="admNav('payments',this)">💰 Payments</button>
      <button class="adm-nav" onclick="admNav('coupons',this)">🎁 Coupons</button>
      <div class="ns">Config</div>
      <button class="adm-nav" onclick="admNav('settings',this)">⚙️ Settings</button>
    </aside>
    <div class="adm-content">
      <div class="adm-page on" id="pg-dashboard">
        <div class="adm-title">Dashboard</div>
        <div class="stat-grid">
          <div class="scard"><div class="sc-icon" style="background:#dbeafe;">💰</div><div class="sc-info"><div class="lbl">Total Revenue</div><div class="val" id="st-rev">₹0</div><div class="chg chg-up">↑ 18% this month</div></div></div>
          <div class="scard"><div class="sc-icon" style="background:#dcfce7;">📦</div><div class="sc-info"><div class="lbl">Total Orders</div><div class="val" id="st-ord">0</div><div class="chg chg-up">↑ 12% this month</div></div></div>
          <div class="scard"><div class="sc-icon" style="background:#fef9c3;">🛍️</div><div class="sc-info"><div class="lbl">Products</div><div class="val" id="st-prd">0</div><div class="chg chg-up">Active listings</div></div></div>
          <div class="scard"><div class="sc-icon" style="background:#fce7f3;">👥</div><div class="sc-info"><div class="lbl">Customers</div><div class="val" id="st-cst">0</div><div class="chg chg-up">↑ 8% this month</div></div></div>
        </div>
        <div class="chart-grid">
          <div class="ch-card"><h3>📈 Revenue – Last 7 Days</h3><div class="bar-chart" id="revChart"></div></div>
          <div class="ch-card"><h3>🥧 Sales by Category</h3><div class="donut-wr" id="catChart"></div></div>
        </div>
        <div class="adm-card"><div class="adm-card-hdr"><span class="adm-card-title">Recent Orders</span></div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th>Order ID</th><th>Customer</th><th>Amount</th><th>Method</th><th>Status</th><th>Date</th></tr></thead><tbody id="dshOrders"></tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-orders">
        <div class="adm-title">Orders <button class="adm-btn" onclick="exportCSV()">⬇ Export CSV</button></div>
        <div class="adm-card">
          <div class="adm-card-hdr">
            <input class="adm-srch" placeholder="Search orders..." oninput="filterOrders(this.value)">
            <select class="sel-sm" onchange="filterOrderStat(this.value)"><option value="">All Status</option><option>pending</option><option>paid</option><option>shipped</option><option>delivered</option><option>cancelled</option></select>
          </div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th>Order ID</th><th>Customer</th><th>Items</th><th>Total</th><th>Method</th><th>Status</th><th>Date</th><th>Action</th></tr></thead><tbody id="ordTbl"></tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-products">
        <div class="adm-title">Products <button class="adm-btn" onclick="openPF(null)">+ Add Product</button></div>
        <div class="adm-card">
          <div class="adm-card-hdr"><input class="adm-srch" placeholder="Search products..." oninput="filterProds(this.value)"></div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th></th><th>Name</th><th>Category</th><th>Price</th><th>MRP</th><th>Disc</th><th>Rating</th><th>Stock</th><th>Actions</th></tr></thead><tbody id="prdTbl"></tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-inventory">
        <div class="adm-title">Inventory</div>
        <div class="adm-card">
          <div class="adm-card-hdr"><input class="adm-srch" placeholder="Search..." oninput="filterInv(this.value)"></div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th>Product</th><th>Category</th><th>Stock</th><th>Status</th><th>Update</th></tr></thead><tbody id="invTbl"></tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-customers">
        <div class="adm-title">Customers</div>
        <div class="adm-card">
          <div class="adm-card-hdr"><input class="adm-srch" placeholder="Search customers..." oninput="filterCusts(this.value)"></div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th>Name</th><th>Email</th><th>Mobile</th><th>City</th><th>Orders</th><th>Spent</th><th>Joined</th></tr></thead><tbody id="custTbl"></tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-sellerreq">
        <div class="adm-title">Seller Requests <span id="sellerReqCount" style="font-size:12px;color:var(--muted);font-weight:600;"></span></div>
        <div class="adm-card">
          <div class="adm-card-hdr">
            <input class="adm-srch" placeholder="Search by name or business..." oninput="filterSellerReq(this.value)">
            <select class="sel-sm" onchange="filterSellerReqStat(this.value)"><option value="">All Status</option><option value="pending">pending</option><option value="accepted">accepted</option><option value="rejected">rejected</option></select>
          </div>
          <div id="sellerReqList" style="padding:6px 0;"></div>
        </div>
      </div>
      <div class="adm-page" id="pg-payments">
        <div class="adm-title">Payments & Transactions</div>
        <div class="stat-grid">
          <div class="scard"><div class="sc-icon" style="background:#dbeafe;">📲</div><div class="sc-info"><div class="lbl">UPI Received</div><div class="val" id="pay-upi">₹0</div></div></div>
          <div class="scard"><div class="sc-icon" style="background:#dcfce7;">💳</div><div class="sc-info"><div class="lbl">Card Payments</div><div class="val" id="pay-card">₹0</div></div></div>
          <div class="scard"><div class="sc-icon" style="background:#f3e5f5;">🏦</div><div class="sc-info"><div class="lbl">Net Banking</div><div class="val" id="pay-net">₹0</div></div></div>
          <div class="scard"><div class="sc-icon" style="background:#fff8e1;">💵</div><div class="sc-info"><div class="lbl">COD Pending</div><div class="val" id="pay-cod">₹0</div></div></div>
        </div>
        <div class="adm-card">
          <div class="adm-card-hdr"><div class="adm-card-title">UPI ID: <b style="color:var(--blue);">9125442370-2@ybl</b></div></div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th>Txn ID</th><th>Order</th><th>Customer</th><th>Amount</th><th>Method</th><th>Status</th><th>Date</th></tr></thead><tbody id="payTbl"></tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-coupons">
        <div class="adm-title">Coupons & Offers</div>
        <div class="adm-card">
          <div class="adm-card-hdr"><div class="adm-card-title">Active Coupons</div></div>
          <div class="adm-tbl-wrap"><table class="atbl"><thead><tr><th>Code</th><th>Type</th><th>Value</th><th>Min Order</th><th>Usage</th><th>Status</th></tr></thead>
          <tbody>
            <tr><td><b>SPIN5</b></td><td>% Discount</td><td>5%</td><td>₹0</td><td>Auto</td><td><span class="sbadge sb-paid">Active</span></td></tr>
            <tr><td><b>SPIN10</b></td><td>% Discount</td><td>10%</td><td>₹0</td><td>Auto</td><td><span class="sbadge sb-paid">Active</span></td></tr>
            <tr><td><b>SPIN15</b></td><td>% Discount</td><td>15%</td><td>₹0</td><td>Auto</td><td><span class="sbadge sb-paid">Active</span></td></tr>
            <tr><td><b>SPIN20</b></td><td>% Discount</td><td>20%</td><td>₹0</td><td>Auto</td><td><span class="sbadge sb-paid">Active</span></td></tr>
            <tr><td><b>FLAT100</b></td><td>Flat ₹ Off</td><td>₹100</td><td>₹500</td><td>Auto</td><td><span class="sbadge sb-paid">Active</span></td></tr>
            <tr><td><b>FREESHIP</b></td><td>Free Shipping</td><td>Free</td><td>₹0</td><td>Auto</td><td><span class="sbadge sb-paid">Active</span></td></tr>
            <tr><td><b>NEW10</b></td><td>% Discount</td><td>10%</td><td>₹200</td><td>New Users</td><td><span class="sbadge sb-paid">Active</span></td></tr>
          </tbody></table></div>
        </div>
      </div>
      <div class="adm-page" id="pg-settings">
        <div class="adm-title">Settings</div>
        <div class="set-grid">
          <div class="set-card"><h3>🏪 Store Info</h3>
            <div class="set-row"><span>Store Name</span><input class="set-input" value="ShopKart"></div>
            <div class="set-row"><span>UPI ID</span><input class="set-input" style="width:200px;font-size:12px;font-family:monospace;" value="9125442370-2@ybl"></div>
            <div class="set-row"><span>Admin Email</span><input class="set-input" style="width:200px;font-size:12px;" value="aksahuakhil@gmail.com"></div>
            <div class="set-row"><span>Currency</span><input class="set-input" value="INR (₹)"></div>
          </div>
          <div class="set-card"><h3>⚙️ Store Toggles</h3>
            <div class="set-row"><span>COD Enabled</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
            <div class="set-row"><span>UPI Payments</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
            <div class="set-row"><span>Spin & Win</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
            <div class="set-row"><span>Sale Banner</span><label class="toggle-wrap"><input type="checkbox" checked><span class="tog-sl"></span></label></div>
            <div class="set-row"><span>Maintenance Mode</span><label class="toggle-wrap"><input type="checkbox"><span class="tog-sl"></span></label></div>
          </div>
          <div class="set-card"><h3>🚚 Delivery Config</h3>
            <div class="set-row"><span>Free delivery above</span><input class="set-input" value="₹500"></div>
            <div class="set-row"><span>Standard fee</span><input class="set-input" value="₹40"></div>
            <div class="set-row"><span>COD handling fee</span><input class="set-input" value="₹20"></div>
            <div class="set-row"><span>Expected days</span><input class="set-input" value="2–5 days"></div>
          </div>
          <div class="set-card"><h3>🔐 Admin Access</h3>
            <div class="set-row"><span>Admin Email</span><b style="font-size:12px;color:var(--blue);">aksahuakhil@gmail.com</b></div>
            <div class="set-row"><span>Password</span><span style="font-size:12px;color:var(--muted);">••••••••</span></div>
            <div class="set-row"><span>Two-Factor Auth</span><label class="toggle-wrap"><input type="checkbox"><span class="tog-sl"></span></label></div>
            <div class="set-row"><span>Session Timeout</span><input class="set-input" value="30 min"></div>
          </div>
        </div>
        <div style="margin-top:14px;"><button class="adm-btn" onclick="saveAllSettings()">Save All Settings</button></div>
      </div>
    </div>
  </div>
</div>

<div class="seller-page" id="sellerPage">
  <div class="seller-top">
    <div class="logo">TRENDRA <sub>SHOPKART</sub></div>
    <button class="seller-close" onclick="closeSellerPage()">← Back to Store</button>
  </div>

  <div class="seller-hero">
    <h1>Grow your business with <span>TRENDRA SHOPKART</span></h1>
    <p>Join thousands of sellers reaching millions of customers across India. Zero setup fee, easy onboarding, and payouts every week.</p>
    <div class="seller-cta-row">
      <button class="seller-cta-primary" onclick="document.getElementById('sellerFormCard').scrollIntoView({behavior:'smooth'})">Start Selling Today →</button>
      <button class="seller-cta-secondary" onclick="document.getElementById('sellerBenefits').scrollIntoView({behavior:'smooth'})">Learn More</button>
    </div>
  </div>

  <div class="seller-stats">
    <div class="seller-stat"><div class="num">5 Cr+</div><div class="lbl">Active Customers</div></div>
    <div class="seller-stat"><div class="num">2 Lakh+</div><div class="lbl">Sellers Onboarded</div></div>
    <div class="seller-stat"><div class="num">15,000+</div><div class="lbl">Pin Codes Covered</div></div>
    <div class="seller-stat"><div class="num">Weekly</div><div class="lbl">Payout Cycle</div></div>
  </div>

  <div class="seller-benefits" id="sellerBenefits">
    <h2>Why sell with us?</h2>
    <div class="seller-benefit-grid">
      <div class="seller-benefit-card"><div class="icon">💰</div><h4>Low Commission</h4><p>Competitive commission rates starting as low as 4%, no hidden charges.</p></div>
      <div class="seller-benefit-card"><div class="icon">🚚</div><h4>Logistics Support</h4><p>We handle pickup, packaging guidance and delivery across India.</p></div>
      <div class="seller-benefit-card"><div class="icon">📊</div><h4>Seller Dashboard</h4><p>Track orders, inventory and earnings in real time from one place.</p></div>
      <div class="seller-benefit-card"><div class="icon">💳</div><h4>Fast Payments</h4><p>Get your payouts deposited directly to your bank every week.</p></div>
      <div class="seller-benefit-card"><div class="icon">🎯</div><h4>Marketing Reach</h4><p>Get featured in deals, banners and personalized recommendations.</p></div>
      <div class="seller-benefit-card"><div class="icon">🛡️</div><h4>Buyer Protection</h4><p>Secure transactions and dispute resolution support for sellers.</p></div>
    </div>
  </div>

  <div class="seller-steps">
    <h2>How it works</h2>
    <div class="seller-step-row">
      <div class="seller-step"><div class="seller-step-num">1</div><h4>Register</h4><p>Fill the form with your business details</p></div>
      <div class="seller-step"><div class="seller-step-num">2</div><h4>Verify</h4><p>Submit GST & bank details for verification</p></div>
      <div class="seller-step"><div class="seller-step-num">3</div><h4>List Products</h4><p>Upload your catalogue with prices & photos</p></div>
      <div class="seller-step"><div class="seller-step-num">4</div><h4>Start Selling</h4><p>Receive orders and grow your business</p></div>
    </div>
  </div>

  <div class="seller-form-wrap">
    <div class="seller-form-card" id="sellerFormCard">
      <div id="sellerFormContent"></div>
    </div>
  </div>
</div>

<script>
const secMap={mobiles:'mobiles',tvs:'tvs',electronics:'electronics',fashion:'fashion',appliances:'appliances',furniture:'furniture',toysbooks:'toys',grocery:'grocery'};
const dealIds=[11,15,17,28,30,31,33,35,2,7];

let orders=[
  {id:'SK20240001',customer:'Rahul Sharma',email:'rahul@gmail.com',mobile:'9876543210',city:'Prayagraj',items:2,total:20798,payment:'upi',status:'delivered',date:'2026-06-01'},
  {id:'SK20240002',customer:'Priya Singh',email:'priya@gmail.com',mobile:'8765432109',city:'Lucknow',items:1,total:22499,payment:'card',status:'shipped',date:'2026-06-05'},
  {id:'SK20240003',customer:'Amit Kumar',email:'amit@gmail.com',mobile:'7654321098',city:'Varanasi',items:3,total:4297,payment:'upi',status:'paid',date:'2026-06-08'},
  {id:'SK20240004',customer:'Sneha Verma',email:'sneha@gmail.com',mobile:'6543210987',city:'Agra',items:1,total:52990,payment:'netbank',status:'delivered',date:'2026-06-10'},
  {id:'SK20240005',customer:'Raj Patel',email:'raj@gmail.com',mobile:'9988776655',city:'Delhi',items:2,total:1898,payment:'cod',status:'pending',date:'2026-06-12'},
  {id:'SK20240006',customer:'Meena Gupta',email:'meena@gmail.com',mobile:'8877665544',city:'Kanpur',items:1,total:24990,payment:'upi',status:'paid',date:'2026-06-13'},
  {id:'SK20240007',customer:'Vikas Joshi',email:'vikas@gmail.com',mobile:'7766554433',city:'Mathura',items:4,total:7095,payment:'card',status:'cancelled',date:'2026-06-14'},
  {id:'SK20240008',customer:'Nisha Yadav',email:'nisha@gmail.com',mobile:'6655443322',city:'Allahabad',items:2,total:19498,payment:'upi',status:'delivered',date:'2026-06-15'}
];

let customers=[
  {name:'Rahul Sharma',email:'rahul@gmail.com',mobile:'9876543210',city:'Prayagraj',orders:3,spent:45600,joined:'2025-01-10'},
  {name:'Priya Singh',email:'priya@gmail.com',mobile:'8765432109',city:'Lucknow',orders:2,spent:29800,joined:'2025-03-22'},
  {name:'Amit Kumar',email:'amit@gmail.com',mobile:'7654321098',city:'Varanasi',orders:5,spent:18200,joined:'2024-11-05'},
  {name:'Sneha Verma',email:'sneha@gmail.com',mobile:'6543210987',city:'Agra',orders:1,spent:52990,joined:'2026-01-14'},
  {name:'Raj Patel',email:'raj@gmail.com',mobile:'9988776655',city:'Delhi',orders:4,spent:12400,joined:'2025-07-30'},
  {name:'Meena Gupta',email:'meena@gmail.com',mobile:'8877665544',city:'Kanpur',orders:2,spent:31200,joined:'2025-09-18'},
  {name:'Vikas Joshi',email:'vikas@gmail.com',mobile:'7766554433',city:'Mathura',orders:1,spent:0,joined:'2026-06-14'},
  {name:'Nisha Yadav',email:'nisha@gmail.com',mobile:'6655443322',city:'Allahabad',orders:3,spent:27900,joined:'2025-05-11'}
];

let cart={}, wishlist=new Set(), appliedCoupon=null, spinUsed=false, currentUser=null, editProdId=null;
const ADMIN_EMAIL='aksahuakhil@gmail.com', ADMIN_PASS='admin@123';
let activeFilters={ratings:new Set()};
let currentPDPId=null;
let reviewsDB={
  1:[{name:'Ankit R.',rating:5,date:'2026-05-12',text:'Camera quality is excellent for this price range, battery easily lasts a full day.'},{name:'Priya S.',rating:4,date:'2026-04-28',text:'Good phone overall, display is bright. Wish charging was a bit faster.'}],
  9:[{name:'Rohit K.',rating:5,date:'2026-06-01',text:'Perfect for coding and daily work, fan noise is minimal even under load.'}],
  19:[{name:'Sunita M.',rating:4,date:'2026-05-20',text:'Cooling is great, a bit noisy at night but overall satisfied.'}]
};
let pendingRating=0;

const byId=id=>products.find(p=>p.id===Number(id));
const disc=p=>Math.round((p.mrp-p.price)/p.mrp*100);
const inr=n=>'₹'+Number(n).toLocaleString('en-IN');
function bgCls(cat){const m={mobiles:'bg-mob',tvs:'bg-tv',electronics:'bg-elec',fashion:'bg-fash',appliances:'bg-app',furniture:'bg-home',toys:'bg-toy',books:'bg-book',grocery:'bg-groc',sports:'bg-spt'};return m[cat]||'bg-mob';}

function pCard(p){
  const d=disc(p),bg=bgCls(p.category);
  const imgHtml=p.photo?`<img src="${p.photo}" style="width:100%;height:100%;object-fit:cover;">`:p.emoji;
  return `<div class="pcard" onclick="openPD(${p.id})">
    <button class="wish" onclick="toggleWish(event,${p.id})">${wishlist.has(p.id)?'❤️':'🤍'}</button>
    ${p.id<=4?'<span class="badge-new">NEW</span>':''}
    <div class="img-wrap ${bg}">${imgHtml}</div>
    <div class="pname-short">${p.name.length>40?p.name.slice(0,40)+'...':p.name}</div>
    <div class="prating"><span class="rating-star">${p.rating}★</span><span class="rating-count">(${p.reviews.toLocaleString('en-IN')})</span></div>
    <div class="pprice"><span class="sp">${inr(p.price)}</span><span class="mrp">${inr(p.mrp)}</span><span class="off">${d}% off</span></div>
    <button class="add-btn" id="ab-${p.id}" onclick="addCart(event,${p.id})">ADD TO CART</button>
  </div>`;
}

function renderSections(){
  const ds=document.getElementById('dealStrip');
  if(ds){
    const dealProducts=dealIds.map(id=>byId(id)).filter(Boolean);
    ds.innerHTML=dealProducts.length?dealProducts.map(p=>`<div class="deal-thumb" onclick="openPD(${p.id})"><div class="dt-img">${p.photo?`<img src="${p.photo}" style="width:100%;height:100%;object-fit:cover;border-radius:6px;">`:p.emoji}</div><div style="font-size:11px;font-weight:600;">${p.name.split(' ').slice(0,3).join(' ')}</div><div class="dt-off">${disc(p)}% off</div><div class="dt-cat">${p.category}</div></div>`).join(''):'<div style="padding:24px;color:var(--muted);font-size:13px;">No deals available right now.</div>';
  }
  for(const[sectionKey,catValue]of Object.entries(secMap)){
    const el=document.getElementById('s-'+sectionKey);
    if(!el)continue;
    const list=products.filter(p=>p.category===catValue);
    const filtered=getFilteredSorted(list);
    if(!list.length){
      el.innerHTML=`<div class="empty-cat-msg">📭 Not available right now in this category.</div>`;
    } else if(!filtered.length){
      el.innerHTML=`<div class="empty-cat-msg">No products match the current filters in this category.</div>`;
    } else {
      el.innerHTML=filtered.map(p=>pCard(p)).join('');
    }
  }
}

/* ─── PRODUCT DETAIL PAGE (dynamic route, not a modal) ──────────────────── */
function openPD(id){
  const p=byId(id);
  if(!p){toast('This product is no longer available');return;}
  currentPDPId=id;
  document.getElementById('homeSec').style.display='none';
  document.getElementById('searchSec').style.display='none';
  document.getElementById('filterBar').style.display='none';
  const pdpSec=document.getElementById('pdpSec');
  pdpSec.style.display='block';
  renderProductDetailPage(id);
  window.scrollTo({top:0,behavior:'smooth'});
}
function closePDP(){
  currentPDPId=null;
  document.getElementById('pdpSec').style.display='none';
  document.getElementById('homeSec').style.display='block';
  document.getElementById('filterBar').style.display='flex';
}
function renderProductDetailPage(id){
  const p=byId(id);
  const pdpSec=document.getElementById('pdpSec');
  if(!p){
    pdpSec.innerHTML=`<div class="empty-cat-msg" style="padding:80px 20px;">😕 This product is no longer available.<br><br><button class="btn-cart" style="width:auto;padding:10px 24px;" onclick="closePDP()">← Back to Store</button></div>`;
    return;
  }
  const d=disc(p),bg=bgCls(p.category);
  const imgHtml=p.photo?`<img src="${p.photo}" style="width:100%;height:100%;object-fit:cover;">`:p.emoji;
  const related=products.filter(x=>x.category===p.category&&x.id!==p.id).slice(0,6);
  pdpSec.innerHTML=`
    <div class="pdp-crumb"><span onclick="closePDP()">Home</span> / <span onclick="scrollSec('${p.category}');closePDP()" style="text-transform:capitalize;">${p.category}</span> / ${p.name.slice(0,40)}</div>
    <div class="pdp-wrap">
      <div class="pdp-left">
        <div class="pdp-img ${bg}">${imgHtml}</div>
        <div class="pdp-actions">
          <button class="btn-cart" onclick="addCart(null,${p.id})">🛒 Add to Cart</button>
          <button class="btn-buy" onclick="addCart(null,${p.id});setTimeout(openCart,100);">⚡ Buy Now</button>
        </div>
      </div>
      <div class="pdp-right">
        <div class="pdp-brand">Brand: <b style="color:var(--text);">${p.brand}</b></div>
        <div class="pdp-title">${p.name}</div>
        <div class="pdp-rating-row">
          <span class="rating-star">${p.rating}★</span>
          <span class="rating-count">${p.reviews.toLocaleString('en-IN')} ratings</span>
          <span style="font-size:12px;color:var(--green);font-weight:700;">${p.stock>10?'✅ In Stock':p.stock>0?'⚠️ Only '+p.stock+' left':'❌ Out of Stock'}</span>
        </div>
        <div class="pdp-price-row">
          <span class="pdp-price">${inr(p.price)}</span>
          <span class="pdp-mrp">${inr(p.mrp)}</span>
          <span class="pdp-off">${d}% off</span>
        </div>
        <div style="color:var(--green);font-size:13px;font-weight:700;">You save ${inr(p.mrp-p.price)}!</div>
        ${p.emi&&p.emi!=='N/A'?`<div class="pd-emi">No-cost EMI from ${p.emi} · Bank offers available</div>`:''}
        <div class="pd-badges">
          <span class="pd-badge">🚚 Free Delivery</span><span class="pd-badge">↩️ 7-Day Return</span>
          <span class="pd-badge">✅ Genuine Product</span>
        </div>
        <div class="pdp-section">
          <h3>Product Description</h3>
          <p style="font-size:13px;color:var(--text);line-height:1.7;">${p.desc}</p>
        </div>
        <div class="pdp-section">
          <h3>Specifications</h3>
          <table class="pdp-spec-table">
            <tr><td>Brand</td><td>${p.brand}</td></tr>
            <tr><td>Category</td><td style="text-transform:capitalize;">${p.category}</td></tr>
            <tr><td>In Stock</td><td>${p.stock} units</td></tr>
          </table>
        </div>
        <div class="pdp-section" id="reviewsBlock-${p.id}"></div>
        ${related.length?`
        <div class="pdp-section">
          <h3>Related Products</h3>
          <div class="pdp-related">${related.map(rp=>pCard(rp)).join('')}</div>
        </div>`:''}
      </div>
    </div>`;
  renderReviewsBlock(id);
}

function getProductReviews(id){return reviewsDB[id]||[];}

function renderReviewsBlock(id){
  const el=document.getElementById('reviewsBlock-'+id);
  if(!el)return;
  const revs=getProductReviews(id);
  const p=byId(id);
  const dist=[5,4,3,2,1].map(star=>revs.filter(r=>r.rating===star).length);
  const maxCount=Math.max(1,...dist);
  el.innerHTML=`
    <div class="pd-spec" style="margin-top:20px;border-top:1px solid var(--border);padding-top:18px;">
      <h4>Ratings & Reviews</h4>
      <div class="review-summary">
        <div>
          <div class="review-big-num">${p.rating}★</div>
          <div class="review-stars-row">${'★'.repeat(Math.round(p.rating))}${'☆'.repeat(5-Math.round(p.rating))}</div>
          <div class="review-count-txt">${p.reviews.toLocaleString('en-IN')} ratings · ${revs.length} reviews</div>
        </div>
        <div class="review-bars">
          ${[5,4,3,2,1].map((star,i)=>`<div class="review-bar-row"><span>${star}★</span><div class="review-bar-track"><div class="review-bar-fill" style="width:${(dist[i]/maxCount*100)}%"></div></div><span>${dist[i]}</span></div>`).join('')}
        </div>
      </div>
      <div class="review-form">
        <div style="font-size:12px;font-weight:700;color:var(--muted);text-transform:uppercase;margin-bottom:6px;">Write a Review</div>
        <div class="review-star-picker" id="reviewStarPicker">${[1,2,3,4,5].map(n=>`<span data-n="${n}" onclick="setPendingRating(${n})">★</span>`).join('')}</div>
        <textarea id="reviewText-${id}" placeholder="Share your experience with this product..."></textarea>
        <button class="btn-cart" style="width:auto;padding:9px 22px;" onclick="submitReview(${id})">Submit Review</button>
      </div>
      <div id="reviewsList-${id}">
        ${revs.length?revs.map(r=>`
          <div class="review-item">
            <div class="review-item-head">
              <div class="review-avatar">${r.name[0]}</div>
              <div><div class="review-item-name">${r.name}</div><div class="review-item-date">${r.date}</div></div>
            </div>
            <div class="review-item-stars">${'★'.repeat(r.rating)}${'☆'.repeat(5-r.rating)}</div>
            <div class="review-item-text">${r.text}</div>
          </div>`).join(''):'<div style="color:var(--muted);font-size:13px;padding:10px 0;">No reviews yet. Be the first to review!</div>'}
      </div>
    </div>`;
}

function setPendingRating(n){
  pendingRating=n;
  document.querySelectorAll('#reviewStarPicker span').forEach(s=>{
    s.classList.toggle('sel',Number(s.dataset.n)<=n);
  });
}

function submitReview(id){
  if(pendingRating===0){toast('Please select a star rating');return;}
  const textEl=document.getElementById('reviewText-'+id);
  const text=textEl?textEl.value.trim():'';
  if(!text){toast('Please write a short review');return;}
  if(!reviewsDB[id])reviewsDB[id]=[];
  const name=currentUser?currentUser.name:'Guest User';
  reviewsDB[id].unshift({name,rating:pendingRating,date:new Date().toISOString().slice(0,10),text});
  const p=byId(id);
  const allRatings=reviewsDB[id].map(r=>r.rating);
  p.rating=Math.round((allRatings.reduce((a,b)=>a+b,0)/allRatings.length)*10)/10;
  p.reviews=p.reviews+1;
  pendingRating=0;
  renderReviewsBlock(id);
  toast('✅ Review submitted, thank you!');
}

function toggleWish(e,id){
  e.stopPropagation();
  wishlist.has(id)?wishlist.delete(id):wishlist.add(id);
  document.getElementById('wishBadge').textContent=wishlist.size;
  toast(wishlist.has(id)?'Added to wishlist ❤️':'Removed from wishlist');
  document.querySelectorAll('.pcard').forEach(c=>{});
  renderSections();renderWishModal();
}
function renderWishModal(){
  const g=document.getElementById('wishGrid');
  document.getElementById('wishCount').textContent=wishlist.size;
  if(!wishlist.size){g.innerHTML='<div style="grid-column:1/-1;text-align:center;padding:40px;color:var(--muted);">No items in wishlist. Start adding! ❤️</div>';return;}
  g.innerHTML=[...wishlist].map(id=>{const p=byId(id);return p?pCard(p):'';}).join('');
}

function addCart(e,id){
  if(e)e.stopPropagation();
  const p=byId(id);
  if(!p){toast('This product is no longer available');renderCart();return;}
  cart[id]=(cart[id]||0)+1;
  updBadge();renderCart();
  const btn=document.getElementById('ab-'+id);
  if(btn){btn.textContent='ADDED ✓';btn.classList.add('done');setTimeout(()=>{btn.textContent='ADD TO CART';btn.classList.remove('done');},1300);}
  toast('Added to cart 🛒');
}
function chgQty(id,d){cart[id]=(cart[id]||0)+d;if(cart[id]<=0)delete cart[id];updBadge();renderCart();}
function remItem(id){delete cart[id];updBadge();renderCart();toast('Item removed');}
function updBadge(){document.getElementById('cartBadge').textContent=Object.values(cart).reduce((a,b)=>a+b,0);}

// FIX: COD fee now included in BOTH cart sidebar total AND checkout total, consistently
function getTotal(){
  let sub=0;
  for(const[id,qty]of Object.entries(cart))sub+=byId(Number(id)).price*qty;
  let d=0;
  if(appliedCoupon){
    if(appliedCoupon.type==='percent')d=Math.round(sub*appliedCoupon.value/100);
    if(appliedCoupon.type==='flat')d=Math.min(appliedCoupon.value,sub);
  }
  const netSub=sub-d;
  const cod=(typeof coPayMethod!=='undefined'&&coPayMethod==='cod')?20:0;
  // FIX: free delivery threshold checked on netSub (after discount) consistently
  const del=(netSub>500||(appliedCoupon&&appliedCoupon.type==='shipping'))?0:40;
  return{sub,disc:d,del,cod,total:netSub+del+cod};
}

function renderCart(){
  const body=document.getElementById('cartBody'),foot=document.getElementById('cartFoot');
  // FIX: auto-remove cart lines for products that no longer exist in Firestore (deleted by admin)
  let removedStale=false;
  Object.keys(cart).forEach(id=>{
    if(!byId(Number(id))){delete cart[id];removedStale=true;}
  });
  if(removedStale){updBadge();toast('Some items in your cart are no longer available and were removed');}
  const ents=Object.entries(cart).filter(([,q])=>q>0);
  if(!ents.length){
    body.innerHTML='<div class="cart-empty"><div class="ce-icon">🛒</div><b>Your cart is empty</b><br>Add items to get started!</div>';
    foot.innerHTML='';return;
  }
  body.innerHTML=ents.map(([id,qty])=>{
    const p=byId(Number(id));
    return `<div class="cart-item">
      <div class="ci-img ${bgCls(p.category)}">${p.emoji}</div>
      <div class="ci-info">
        <div class="ci-name">${p.name}</div>
        <div class="ci-price">${inr(p.price*qty)}</div>
        <div class="ci-qty"><button onclick="chgQty(${p.id},-1)">−</button><span>${qty}</span><button onclick="chgQty(${p.id},1)">+</button></div>
        <button class="ci-remove" onclick="remItem(${p.id})">REMOVE</button>
      </div>
    </div>`;
  }).join('');
  // FIX: cart sidebar now shows COD fee line item too, so total in sidebar matches checkout
  const{sub,disc:d,del,cod,total}=getTotal();
  foot.innerHTML=`
    ${appliedCoupon?`<div class="coupon-strip"><span>🎁 ${appliedCoupon.label} (${appliedCoupon.code})</span><span style="cursor:pointer;" onclick="remCoupon()">✕</span></div>`:''}
    <div class="sum-row"><span>Subtotal (${Object.values(cart).reduce((a,b)=>a+b,0)} items)</span><span>${inr(sub)}</span></div>
    ${d>0?`<div class="sum-row"><span>Coupon Discount</span><span style="color:var(--green);font-weight:700;">−${inr(d)}</span></div>`:''}
    <div class="sum-row"><span>Delivery</span><span>${del===0?'<span style="color:var(--green);font-weight:700;">FREE</span>':inr(del)}</span></div>
    ${cod>0?`<div class="sum-row"><span>COD Fee</span><span>${inr(cod)}</span></div>`:''}
    <div class="sum-row total"><span>Total Amount</span><span>${inr(total)}</span></div>
    <button class="cart-cta" onclick="startCheckout()">PLACE ORDER →</button>`;
}
function remCoupon(){appliedCoupon=null;renderCart();toast('Coupon removed');}

function openCart(){document.getElementById('cartSide').classList.add('on');document.getElementById('overlay').classList.add('on');}
function closeCart(){document.getElementById('cartSide').classList.remove('on');document.getElementById('overlay').classList.remove('on');}

function openM(id){document.querySelectorAll('.modal').forEach(m=>m.classList.remove('on'));document.getElementById(id).classList.add('on');document.getElementById('overlay').classList.add('on');}
function closeM(id){document.getElementById(id).classList.remove('on');document.getElementById('overlay').classList.remove('on');}
function closeAll(){document.querySelectorAll('.modal').forEach(m=>m.classList.remove('on'));document.getElementById('cartSide').classList.remove('on');document.getElementById('overlay').classList.remove('on');}

/* ─── TRENDRA AI — Hindi/English natural-language product finder ──────────── */
let baristaInit=false;
let baristaHistory=[];

const AI_COLOR_WORDS={
  kala:'black',kali:'black',kale:'black',black:'black',
  safed:'white',white:'white',
  laal:'red',lal:'red',red:'red',
  neela:'blue',nila:'blue',blue:'blue',
  hara:'green',green:'green',
  pila:'yellow',peela:'yellow',yellow:'yellow'
};
const AI_CATEGORY_WORDS={
  pant:'fashion',pants:'fashion',pajama:'fashion',jeans:'fashion',trouser:'fashion',trousers:'fashion',
  shirt:'fashion',kurti:'fashion',dress:'fashion',jacket:'fashion',tshirt:'fashion','t-shirt':'fashion',
  kapde:'fashion',kapda:'fashion',kapdo:'fashion',cloth:'fashion',clothes:'fashion',
  mobile:'mobiles',phone:'mobiles',smartphone:'mobiles',
  tv:'tvs',television:'tvs',
  laptop:'electronics',earbuds:'electronics',speaker:'electronics',
  fridge:'appliances',ac:'appliances',washingmachine:'appliances',
  sofa:'furniture',table:'furniture',
  toy:'toys',book:'books',
  rice:'grocery',oil:'grocery'
};
const AI_CHEAP_WORDS=['sasta','cheap','low price','kam price','budget'];
const AI_PREMIUM_WORDS=['premium','best','top','expensive','mehenga','mahenga'];

function aiParseQuery(q){
  const lower=q.toLowerCase();
  const words=lower.split(/\s+/).filter(Boolean);
  let color=null,category=null,sortCheap=false,sortPremium=false;
  for(const w of words){
    if(AI_COLOR_WORDS[w])color=AI_COLOR_WORDS[w];
    if(AI_CATEGORY_WORDS[w])category=AI_CATEGORY_WORDS[w];
  }
  if(AI_CHEAP_WORDS.some(p=>lower.includes(p)))sortCheap=true;
  if(AI_PREMIUM_WORDS.some(p=>lower.includes(p)))sortPremium=true;
  return{color,category,sortCheap,sortPremium,raw:lower,words};
}

function aiSearchProducts(query){
  const parsed=aiParseQuery(query);
  let results=products.filter(p=>{
    const nameLower=p.name.toLowerCase();
    let ok=true;
    if(parsed.color)ok=ok&&nameLower.includes(parsed.color);
    if(parsed.category)ok=ok&&p.category===parsed.category;
    return ok;
  });
  // fallback: if no structured match, try plain keyword search across name/brand/category
  if(!results.length&&!parsed.color&&!parsed.category){
    results=products.filter(p=>parsed.words.some(w=>w.length>2&&(p.name.toLowerCase().includes(w)||p.brand.toLowerCase().includes(w)||p.category.includes(w))));
  }
  if(parsed.sortCheap)results=results.slice().sort((a,b)=>a.price-b.price);
  if(parsed.sortPremium)results=results.slice().sort((a,b)=>b.price-a.price);
  return results.slice(0,6);
}

function baristaBubble(html,who){
  return `<div class="bar-msg ${who}">${html}</div>`;
}

function aiProductMiniCard(p){
  const d=disc(p);
  const imgHtml=p.photo?`<img src="${p.photo}" style="width:100%;height:100%;object-fit:cover;border-radius:6px;">`:p.emoji;
  return `<div style="display:flex;gap:10px;align-items:center;background:rgba(255,255,255,.06);border-radius:8px;padding:8px;margin-top:6px;cursor:pointer;" onclick="closeM('baristaModal');openPD(${p.id})">
    <div style="width:44px;height:44px;border-radius:6px;background:var(--card);display:flex;align-items:center;justify-content:center;font-size:22px;flex-shrink:0;">${imgHtml}</div>
    <div style="flex:1;min-width:0;">
      <div style="font-size:12px;font-weight:600;color:var(--text);white-space:nowrap;overflow:hidden;text-overflow:ellipsis;">${p.name}</div>
      <div style="font-size:12px;font-weight:700;color:var(--text);">${inr(p.price)} <span style="font-size:10px;color:var(--green);font-weight:600;">${d}% off</span></div>
    </div>
    <button class="bm-addbtn" onclick="event.stopPropagation();addCart(event,${p.id});toast('Added to cart 🛒');">ADD</button>
  </div>`;
}

function initBarista(){
  if(baristaInit)return;
  baristaInit=true;
  const chat=document.getElementById('barChat');
  chat.innerHTML=baristaBubble(`Hi! Main Trendra AI hoon 🤖<br>Product dhundne ke liye type karo — jaise <b>"kala pant"</b>, <b>"red dress"</b>, ya <b>"sasta mobile"</b>.<br>Order, return, delivery jaisa sawaal bhi pooch sakte ho!`,'bot');
}

function trendraSystemPrompt(){
  const catalogue = products.map(p=>({id:p.id,name:p.name,category:p.category,price:p.price,brand:p.brand,desc:p.desc})).slice(0,300);
  const categories = [...new Set(products.map(p=>p.category))];
  const CATEGORY_LABELS={mobiles:'Mobiles',tvs:'TVs',electronics:'Electronics',fashion:'Fashion',appliances:'Appliances',furniture:'Furniture',toys:'Toys',books:'Books',grocery:'Grocery'};
  const categoryLabels = categories.map(c=>CATEGORY_LABELS[c]||c);
  return `Aap "Trendra AI" hain — Trendra Shopkart (ek online shopping website) ka AI shopping assistant.
Aapka kaam: user ki Hindi/Hinglish/English query samajh ke neeche diye gaye PRODUCT CATALOGUE se sabse relevant products dhundna, aur agar koi general sawaal (order status, return policy, shipping, payment) ho to short helpful jawab dena.
Pichle conversation ka context yaad rakho — agar user follow-up bole ("aur sasta dikhao", "blue mein hai kya"), to pichle message ke topic/category ke hisaab se samjho.

AVAILABLE CATEGORIES: ${categoryLabels.join(', ')}

Casual/greeting messages ("hi", "hello", "how are u", "kaise ho") ke liye special behavior:
- Pehle ek warm, friendly reply do (Trendra AI ke roop mein).
- Phir batao ki Trendra Shopkart par yeh categories available hain (upar di gayi list se, natural Hindi/Hinglish mein likho, products mat dikhao abhi).
- Phir poocho ki user kis category mein interested hai — ek hi sawaal poocho, sab categories ek saath mat dikhao expecting answer, sirf list bata ke poocho "kya dekhna chahenge?".
- productIds is turn par empty rakho — pehle category-interest poocho, products tab dikhao jab user koi specific category/item bata de.
- Jab user koi category ya product type bata de uske jawab mein, tab us category ke kuch (3-5) top/popular items dikhao productIds mein, aur reply mein poocho ki inmein se koi pasand aaya ya aur kuch specific (color/price/brand) chahiye — isse ek guided, ek-ek-karke conversation banta hai, sab kuch ek baar mein dump nahi karna.

Rules:
- Hamesha sirf ek valid JSON object return karo, kuch aur text nahi, koi markdown code-fence nahi.
- JSON shape: {"reply": "<chhota, friendly Hindi/Hinglish jawab, max 2-3 lines>", "productIds": [<matching product id numbers from catalogue — hamesha numbers, strings nahi — max 6, sirf wahi jo really relevant hain, empty array agar greeting hai ya koi product query nahi hai>]}
- Sirf catalogue mein diye gaye ids use karo, naye products mat banao.
- name aur desc dono fields use karke match karo — sirf naam mein word dhundna kaafi nahi, matlab/context bhi samjho (jaise "office ke kapde" → formal shirts/trousers, "garmi mein pehenne wala" → cotton/breathable items).
- Agar user color bole (kala/black, laal/red, neela/blue, hara/green, safed/white, pila/yellow), to sirf usi color ke products dikhao jinke naam mein wo color word ho.
- Agar "sasta"/"cheap"/"budget" bole to saste products pehle dikhao; "premium"/"best"/"mehenga" bole to costly/best-rated pehle.
- Spelling mistakes, alag tarike se likhe gaye words (jaise "pant"/"pent", "shrt"/"shirt"), aur mixed Hindi-English ko samajhne ki koshish karo — exact spelling match zaroori nahi hai.
- Agar koi product match nahi karta, productIds empty rakho aur reply mein politely batao.
- Agar order status, return, refund, delivery time, payment options jaisa general sawaal ho, productIds empty rakho aur reply mein concise generic Trendra Shopkart policy info do (free returns within 7 days, COD available, standard delivery 3-5 days, ya jo bhi reasonable e-commerce default hai) — disclaim mat karo, seedha helpful jawab do.

PRODUCT CATALOGUE (JSON array):
${JSON.stringify(catalogue)}`;
}

async function sendBarista(){
  const input=document.getElementById('barInput');
  const q=input.value.trim();
  if(!q)return;
  const chat=document.getElementById('barChat');
  chat.insertAdjacentHTML('beforeend',baristaBubble(q,'user'));
  input.value='';
  chat.scrollTop=chat.scrollHeight;

  const thinkingId='think-'+Date.now();
  chat.insertAdjacentHTML('beforeend',`<div class="bar-msg bot" id="${thinkingId}">Trendra AI soch raha hai...</div>`);
  chat.scrollTop=chat.scrollHeight;

  let replyHtml=null;
  let usedGemini=false;

  // Try real Gemini-powered answer first (needs Firebase AI Logic module to have loaded ok)
  try{
    await window.__trendraAIReady;
    if(typeof window.__trendraAskGemini==='function'){
      baristaHistory.push({role:'user',text:q});
      const raw=await window.__trendraAskGemini(trendraSystemPrompt(), baristaHistory);
      const cleaned=raw.replace(/^```json\s*|\s*```$/g,'').trim();
      const parsed=JSON.parse(cleaned);
      const ids=Array.isArray(parsed.productIds)?parsed.productIds.map(Number).filter(n=>!isNaN(n)):[];
      const matched=ids.map(id=>byId(id)).filter(Boolean);
      replyHtml=(parsed.reply||'')+(matched.length?matched.map(aiProductMiniCard).join(''):'');
      baristaHistory.push({role:'model',text:raw});
      // keep history bounded so the prompt doesn't grow unbounded over a long chat
      if(baristaHistory.length>16)baristaHistory=baristaHistory.slice(-16);
      usedGemini=true;
    }
  }catch(err){
    console.warn('Trendra AI (Gemini) unavailable, using local fallback:',err);
    // don't poison history with a failed turn
    if(baristaHistory.length&&baristaHistory[baristaHistory.length-1].role==='user')baristaHistory.pop();
  }

  // Fallback: local keyword-matching engine (works even without Firebase/Gemini)
  if(!usedGemini){
    const results=aiSearchProducts(q);
    if(results.length){
      replyHtml=`Yeh mila aapke liye:`+results.map(aiProductMiniCard).join('');
    }else{
      replyHtml=`Maaf kijiye, "${q}" ke liye koi product nahi mila. Kuch alag try karein — jaise color (kala/red/blue) ya category (pant/shirt/mobile) ke saath.`;
    }
  }

  document.getElementById(thinkingId)?.remove();
  chat.insertAdjacentHTML('beforeend',baristaBubble(replyHtml,'bot'));
  chat.scrollTop=chat.scrollHeight;
}



function lTab(t){
  const isL=t==='login';
  document.getElementById('tLogin').classList.toggle('act',isL);
  document.getElementById('tSignup').classList.toggle('act',!isL);
  document.getElementById('loginFields').style.display=isL?'block':'none';
  document.getElementById('signupFields').style.display=isL?'none':'block';
  document.getElementById('otpSentPanel').style.display='none';
  document.getElementById('loginDivider').style.display=isL?'flex':'none';
  document.getElementById('loginSocialBtns').style.display=isL?'flex':'none';
  document.getElementById('loginTitle').textContent=isL?'Login':'Create Account';
}
function fillAdmin(){document.getElementById('lemail').value=ADMIN_EMAIL;document.getElementById('lpass').value=ADMIN_PASS;toast('Admin credentials filled ✓');}

let signupOtpMethod='email';
function setOtpMethod(m){
  if(m==='mobile')return; // disabled until Blaze plan + SMS billing is set up
  signupOtpMethod=m;
  document.getElementById('otpChoiceEmail').classList.toggle('act',m==='email');
  document.getElementById('otpChoiceMobile').classList.toggle('act',m==='mobile');
  document.getElementById('smobileRow').style.display=m==='mobile'?'block':'none';
}

// Real Firebase email/password login.
async function doLogin(){
  const email=document.getElementById('lemail').value.trim();
  const pass=document.getElementById('lpass').value.trim();
  if(!email||!pass){toast('Enter email and password');return;}
  if(email.toLowerCase()===ADMIN_EMAIL.toLowerCase()){
    if(pass!==ADMIN_PASS){toast('❌ Wrong admin password');return;}
    currentUser={email:ADMIN_EMAIL,name:'Akhil',isAdmin:true};
    closeM('loginModal');updateHdrUser();
    toast(`Welcome back, ${currentUser.name}! 👋`);
    return;
  }
  try{
    await window.__trendraAIReady;
    if(!window.__trendraAuth) throw new Error('Auth not ready');
    const user=await window.__trendraAuth.signIn(email,pass);
    currentUser={...user,isAdmin:false};
    closeM('loginModal');updateHdrUser();
    toast(`Welcome back, ${currentUser.name}! 👋`);
  }catch(err){
    console.error('Login failed:',err);
    const code=err?.code||'';
    if(code.includes('invalid-credential')||code.includes('wrong-password')||code.includes('user-not-found')){
      toast('❌ Incorrect email or password');
    }else{
      toast('❌ Login failed — please try again');
    }
  }
}

// Real Firebase signup: creates the account, then sends a free email sign-in link
// (Firebase's no-cost "email OTP" equivalent) that the user must click to verify
// before they're considered fully signed in.
async function doSignupSendOtp(){
  const name=document.getElementById('sname').value.trim();
  const email=document.getElementById('semail').value.trim();
  const pass=document.getElementById('spass').value.trim();
  if(!name){toast('Enter your full name');return;}
  if(!email){toast('Enter your email');return;}
  if(email.toLowerCase()===ADMIN_EMAIL.toLowerCase()){toast('This email is reserved — please use Login instead');return;}
  if(!pass||pass.length<6){toast('Password must be at least 6 characters');return;}
  if(signupOtpMethod==='mobile'){toast('Mobile OTP is coming soon — please use Email OTP for now');return;}

  const btn=document.getElementById('signupBtn');
  btn.disabled=true;btn.textContent='SENDING...';
  try{
    await window.__trendraAIReady;
    if(!window.__trendraAuth) throw new Error('Auth not ready');
    await window.__trendraAuth.signUp(email,pass,name);
    await window.__trendraAuth.sendEmailOtp(email);
    window.localStorage.setItem('trendraPendingName',name);
    document.getElementById('signupFields').style.display='none';
    document.getElementById('otpSentPanel').style.display='block';
    document.getElementById('otpSentMsg').textContent=`We've sent a verification link to ${email}. Open it on this device to finish signing in.`;
    document.getElementById('loginDivider').style.display='none';
    document.getElementById('loginSocialBtns').style.display='none';
  }catch(err){
    console.error('Signup failed:',err);
    const code=err?.code||'';
    if(code.includes('email-already-in-use')){
      toast('This email is already registered — try logging in instead');
    }else if(code.includes('invalid-email')){
      toast('Enter a valid email address');
    }else{
      toast('❌ Could not send verification link — please try again');
    }
  }finally{
    btn.disabled=false;btn.textContent='SEND VERIFICATION LINK';
  }
}

// Runs once on page load: if the URL is a Firebase email sign-in link the user just
// clicked from their inbox, complete the sign-in automatically.
async function checkEmailOtpOnLoad(){
  try{
    await window.__trendraAIReady;
    if(!window.__trendraAuth) return;
    const user=await window.__trendraAuth.completeEmailOtpIfPresent();
    if(user){
      const pendingName=window.localStorage.getItem('trendraPendingName');
      currentUser={...user,name:pendingName||user.name,isAdmin:false};
      window.localStorage.removeItem('trendraPendingName');
      updateHdrUser();
      toast(`✅ Email verified! Welcome, ${currentUser.name} 🎉`);
    }
  }catch(err){
    console.error('Email OTP verification failed:',err);
    toast('❌ Verification link is invalid or expired — please sign up again');
  }
}
checkEmailOtpOnLoad();

function socialLogin(p){toast(`Signing in with ${p}...`);setTimeout(()=>{currentUser={email:`user@${p.toLowerCase()}.com`,name:p+'User',isAdmin:false};updateHdrUser();toast('Logged in successfully! 👋');},800);}
function doLogout(){currentUser=null;updateHdrUser();document.getElementById('profileDrop').classList.remove('on');toast('Logged out');}
function updateHdrUser(){
  const area=document.getElementById('hdrUserArea');
  if(!currentUser){area.innerHTML='<button class="hdr-btn" onclick="openM(\'loginModal\')">Login</button>';return;}
  area.innerHTML=`<div style="display:flex;align-items:center;gap:10px;">
    <div class="hdr-avatar" onclick="toggleProfile()" style="cursor:pointer;">${currentUser.name[0].toUpperCase()}</div>
    ${currentUser.isAdmin?'<button style="background:rgba(255,255,255,.2);color:#fff;padding:6px 12px;border-radius:3px;font-weight:600;font-size:12px;" onclick="openAdmin()">⚙️ Admin</button>':''}
  </div>`;
  document.getElementById('pdAvatar').textContent=currentUser.name[0].toUpperCase();
  document.getElementById('pdName').textContent=currentUser.name;
  document.getElementById('pdEmail').textContent=currentUser.email;
  document.getElementById('adminPanelLink').style.display=currentUser.isAdmin?'block':'none';
}
function toggleProfile(){document.getElementById('profileDrop').classList.toggle('on');}
function toggleNotif(){document.getElementById('notifPanel').classList.toggle('on');document.getElementById('profileDrop').classList.remove('on');}
function closeNotif(){document.getElementById('notifPanel').classList.remove('on');}

function renderOrdersModal(){
  const c=document.getElementById('ordersContent');
  const myOrders=orders.slice(0,5);
  if(!myOrders.length){c.innerHTML='<div style="text-align:center;padding:40px;color:var(--muted);">No orders yet.</div>';return;}
  c.innerHTML=myOrders.map(o=>`<div class="order-card">
    <div class="oc-head"><span class="oc-id">${o.id}</span><span class="oc-status st-${o.status}">${o.status.toUpperCase()}</span></div>
    <div class="oc-items">${o.items} item(s) · ${o.payment.toUpperCase()}</div>
    <div class="oc-total">${inr(o.total)} · ${o.date}</div>
  </div>`).join('');
}

let searchTimer=null;
function doSearch(){
  const q=document.getElementById('searchQ').value.trim().toLowerCase();
  const sr=document.getElementById('searchSec'),hs=document.getElementById('homeSec');
  if(!q){sr.style.display='none';hs.style.display='block';return;}
  let m=products.filter(p=>p.name.toLowerCase().includes(q)||p.category.toLowerCase().includes(q)||p.desc.toLowerCase().includes(q)||p.brand.toLowerCase().includes(q));
  m=getFilteredSorted(m);
  document.getElementById('searchSub').textContent=`${m.length} result(s) for "${document.getElementById('searchQ').value.trim()}"`;
  document.getElementById('srGrid').innerHTML=m.length?m.map(p=>pCard(p)).join(''):'<div style="grid-column:1/-1;text-align:center;padding:40px;color:var(--muted);font-size:16px;">😕 No products found.</div>';
  sr.style.display='block';hs.style.display='none';
}
function clearSearch(){document.getElementById('searchQ').value='';document.getElementById('searchSec').style.display='none';document.getElementById('homeSec').style.display='block';}
function scrollSec(id){const el=document.getElementById(id);if(el)el.scrollIntoView({behavior:'smooth'});}
function goHome(){clearSearch();window.scrollTo({top:0,behavior:'smooth'});}

function populateBrandFilter(){
  const sel=document.getElementById('brandSelect');
  if(!sel)return;
  const brands=[...new Set(products.map(p=>p.brand))].sort();
  sel.innerHTML='<option value="">All Brands</option>'+brands.map(b=>`<option value="${b}">${b}</option>`).join('');
}

function toggleRatingChip(el){
  const r=Number(el.dataset.rating);
  document.querySelectorAll('.filter-chip').forEach(c=>{if(c!==el)c.classList.remove('act');});
  if(activeFilters.ratings.has(r)){
    activeFilters.ratings.clear();
    el.classList.remove('act');
  } else {
    activeFilters.ratings.clear();
    activeFilters.ratings.add(r);
    el.classList.add('act');
  }
  applyFilters();
}

function clearFilters(){
  document.getElementById('sortSelect').value='relevance';
  document.getElementById('priceMin').value='';
  document.getElementById('priceMax').value='';
  document.getElementById('brandSelect').value='';
  activeFilters.ratings.clear();
  document.querySelectorAll('.filter-chip').forEach(c=>c.classList.remove('act'));
  applyFilters();
}

function getFilteredSorted(list){
  let out=[...list];
  const min=Number(document.getElementById('priceMin')?.value)||0;
  const max=Number(document.getElementById('priceMax')?.value)||Infinity;
  const brand=document.getElementById('brandSelect')?.value||'';
  const sort=document.getElementById('sortSelect')?.value||'relevance';
  out=out.filter(p=>p.price>=min&&p.price<=max);
  if(brand)out=out.filter(p=>p.brand===brand);
  if(activeFilters.ratings.size){
    const minR=[...activeFilters.ratings][0];
    out=out.filter(p=>p.rating>=minR);
  }
  if(sort==='price_low')out.sort((a,b)=>a.price-b.price);
  else if(sort==='price_high')out.sort((a,b)=>b.price-a.price);
  else if(sort==='rating')out.sort((a,b)=>b.rating-a.rating);
  else if(sort==='newest')out.sort((a,b)=>b.id-a.id);
  return out;
}

function applyFilters(){
  const isSearchActive=document.getElementById('searchSec').style.display!=='none';
  if(isSearchActive){doSearch();return;}
  renderSections();
}

let hIdx=0;
function heroSlide(d){
  const slides=document.querySelectorAll('.hero-slide'),dots=document.querySelectorAll('.hero-dot');
  slides[hIdx].classList.remove('active');dots[hIdx].classList.remove('active');
  hIdx=(hIdx+d+slides.length)%slides.length;
  slides[hIdx].classList.add('active');dots[hIdx].classList.add('active');
}
function initHero(){
  const slides=document.querySelectorAll('.hero-slide'),dc=document.getElementById('heroDots');
  dc.innerHTML=[...slides].map((_,i)=>`<div class="hero-dot${i===0?' active':''}" onclick="heroGo(${i})"></div>`).join('');
  setInterval(()=>heroSlide(1),4000);
}
function heroGo(i){if(i!==hIdx)heroSlide(i-hIdx);}

let coStep=1,coAddr={},coPayMethod='card';

// FIX: checkout now requires login before opening — guest orders no longer dangle
function startCheckout(){
  if(!Object.keys(cart).length){toast('Cart is empty!');return;}
  if(!currentUser){
    toast('Please login to continue checkout');
    closeCart();
    openM('loginModal');
    return;
  }
  closeCart();coStep=1;renderCoStep();openM('coModal');
}
function setCoStep(n){
  for(let i=1;i<=3;i++)document.getElementById('cs'+i).className='co-step'+(i<n?' done':i===n?' act':'');
  document.getElementById('cl1').className='co-line'+(n>1?' done':'');
  document.getElementById('cl2').className='co-line'+(n>2?' done':'');
}
function renderCoStep(){
  setCoStep(coStep);
  const{sub,disc:d,del,cod,total}=getTotal();
  const b=document.getElementById('coBody');
  if(coStep===1){
    b.innerHTML=`
      <h3 style="font-size:14px;font-weight:700;margin-bottom:12px;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;">📦 Delivery Address</h3>
      <label class="co-label">Full Name</label><input class="co-input" id="aName" placeholder="Full name" value="${coAddr.name||''}">
      <label class="co-label">Mobile Number</label><input class="co-input" id="aMob" placeholder="10-digit number" maxlength="10" type="tel" value="${coAddr.mob||''}">
      <label class="co-label">Flat / House No., Building</label><input class="co-input" id="aL1" placeholder="House no., building" value="${coAddr.l1||''}">
      <label class="co-label">Area / Street / Locality</label><input class="co-input" id="aL2" placeholder="Street, locality" value="${coAddr.l2||''}">
      <div class="co-2col">
        <div><label class="co-label">City</label><input class="co-input" id="aCity" value="${coAddr.city||'Prayagraj'}"></div>
        <div><label class="co-label">Pincode</label><input class="co-input" id="aPin" placeholder="6-digit" maxlength="6" value="${coAddr.pin||''}"></div>
      </div>
      <label class="co-label">State</label>
      <select class="co-input" id="aState"><option>Uttar Pradesh</option><option>Maharashtra</option><option>Delhi</option><option>Karnataka</option><option>Tamil Nadu</option><option>Gujarat</option><option>Rajasthan</option><option>West Bengal</option><option>Bihar</option><option>Other</option></select>
      <button class="co-next blue" onclick="coNext1()">Continue to Payment →</button>`;
  } else if(coStep===2){
    b.innerHTML=`
      <h3 style="font-size:14px;font-weight:700;margin-bottom:12px;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;">💳 Select Payment Method</h3>
      <div class="pm-opts">
        <div class="pm-opt ${coPayMethod==='card'?'act':''}" onclick="selPay('card')"><span class="pmi">💳</span>Card</div>
        <div class="pm-opt ${coPayMethod==='netbank'?'act':''}" onclick="selPay('netbank')"><span class="pmi">🏦</span>Net Banking</div>
        <div class="pm-opt ${coPayMethod==='cod'?'act':''}" onclick="selPay('cod')"><span class="pmi">💵</span>COD</div>
      </div>
      <div class="card-panel${coPayMethod==='card'?' on':''}" id="cardPanel">
        <label class="co-label">Card Number</label><input class="co-input" id="cNum" placeholder="XXXX XXXX XXXX XXXX" maxlength="19" oninput="fmtCard(this)">
        <label class="co-label">Name on Card</label><input class="co-input" id="cName" placeholder="Full name">
        <div class="co-2col">
          <div><label class="co-label">Expiry</label><input class="co-input" id="cExp" placeholder="MM / YY" maxlength="7" oninput="fmtExp(this)"></div>
          <div><label class="co-label">CVV</label><input class="co-input" type="password" id="cCVV" placeholder="•••" maxlength="3"></div>
        </div>
        <button class="co-next" onclick="payCard()">💳 Pay ${inr(total)}</button>
      </div>
      <div class="nb-panel${coPayMethod==='netbank'?' on':''}" id="nbPanel">
        <label class="co-label">Select Bank</label>
        <select class="co-input" id="nbBank"><option value="">-- Choose Bank --</option><option>State Bank of India</option><option>HDFC Bank</option><option>ICICI Bank</option><option>Axis Bank</option><option>Kotak Mahindra Bank</option><option>Punjab National Bank</option><option>Bank of Baroda</option><option>Canara Bank</option></select>
        <button class="co-next blue" onclick="payNetBank()">🏦 Proceed to Bank → (${inr(total)})</button>
      </div>
      <div class="cod-panel${coPayMethod==='cod'?' on':''}" id="codPanel">
        <div class="cod-notice">📦 <b>Cash on Delivery</b><br>Pay ${inr(total)} (includes ₹20 COD fee) when your order arrives at your doorstep.</div>
        <button class="co-next" style="margin-top:10px;" onclick="confirmOrder('cod')">✅ Confirm COD Order</button>
      </div>
      <button class="co-back" onclick="goToStep1()">← Back to Address</button>`;
  }
}
// FIX: back button now lets customer return to address step from payment step
function goToStep1(){coStep=1;renderCoStep();}

// FIX: selecting payment method now re-renders the whole step so amounts (esp. COD total) are always fresh
function selPay(m){
  coPayMethod=m;
  renderCoStep();
}
function coNext1(){
  const nm=document.getElementById('aName').value.trim();
  const mb=document.getElementById('aMob').value.trim();
  const l1=document.getElementById('aL1').value.trim();
  const pn=document.getElementById('aPin').value.trim();
  if(!nm||!mb||!l1||!pn){toast('Please fill all required fields');return;}
  if(!/^\d{10}$/.test(mb)){toast('Enter valid 10-digit mobile');return;}
  if(!/^\d{6}$/.test(pn)){toast('Enter valid 6-digit pincode');return;}
  coAddr={name:nm,mob:mb,l1,l2:document.getElementById('aL2').value,city:document.getElementById('aCity').value,pin:pn};
  coStep=2;renderCoStep();
}
function fmtCard(el){let v=el.value.replace(/\D/g,'').substring(0,16);el.value=v.replace(/(.{4})/g,'$1 ').trim();}
function payCard(){
  const n=document.getElementById('cNum').value.replace(/\s/g,'');
  const nm=document.getElementById('cName').value.trim();
  const exp=document.getElementById('cExp').value.trim();
  const cvv=document.getElementById('cCVV').value.trim();
  if(!n||!nm||!exp||!cvv){toast('Fill all card details');return;}
  if(n.length<16){toast('Invalid card number');return;}
  toast('🔄 Processing payment...');setTimeout(()=>confirmOrder('card'),1800);
}
function payNetBank(){const b=document.getElementById('nbBank').value;if(!b){toast('Select your bank');return;}toast(`🏦 Redirecting to ${b}...`);setTimeout(()=>confirmOrder('netbank'),2000);}
function fmtExp(el){let v=el.value.replace(/\D/g,'');if(v.length>=2)v=v.substring(0,2)+' / '+v.substring(2,4);el.value=v;}
function confirmOrder(method){
  const oid='SK'+Date.now().toString().slice(-8);
  const{total}=getTotal();
  const ml={card:'Credit/Debit Card',netbank:'Net Banking',cod:'Cash on Delivery'};
  orders.unshift({id:oid,customer:coAddr.name||'Customer',email:currentUser?currentUser.email:'guest@shop.com',mobile:coAddr.mob||'',city:coAddr.city||'',items:Object.values(cart).reduce((a,b)=>a+b,0),total,payment:method,status:method==='cod'?'pending':'paid',date:new Date().toISOString().slice(0,10)});
  cart={};appliedCoupon=null;coPayMethod='card';updBadge();renderCart();
  document.getElementById('orderIdEl').textContent='ORDER #'+oid;
  document.getElementById('orderPayInfo').textContent='Payment: '+ml[method]+' · Total: '+inr(total);
  closeM('coModal');openM('orderOkModal');
}

const segs=[
  {label:'5% OFF',type:'percent',value:5,code:'SPIN5'},
  {label:'Try Again',type:'none',value:0,code:''},
  {label:'10% OFF',type:'percent',value:10,code:'SPIN10'},
  {label:'Free Ship',type:'shipping',value:0,code:'FREESHIP'},
  {label:'15% OFF',type:'percent',value:15,code:'SPIN15'},
  {label:'₹100 OFF',type:'flat',value:100,code:'FLAT100'},
  {label:'20% OFF',type:'percent',value:20,code:'SPIN20'},
  {label:'Try Again',type:'none',value:0,code:''}
];
function doSpin(){
  if(spinUsed)return;
  spinUsed=true;
  const btn=document.getElementById('spinBtn');
  btn.disabled=true;btn.textContent='SPINNING...';
  const idx=Math.floor(Math.random()*segs.length);
  const rot=360*6-(idx*45+22.5);
  document.getElementById('wheel').style.transform=`rotate(${rot}deg)`;
  setTimeout(()=>{
    const r=segs[idx];
    const el=document.getElementById('spinRes');
    // FIX: "Try Again" now shows neutral grey, only real wins show green
    if(r.type==='none'){
      el.className='spin-res on lost';
      el.innerHTML=`😅 ${r.label}! Better luck next visit.`;
    } else {
      el.className='spin-res on won';
      window._wonCoupon={code:r.code,type:r.type,value:r.value,label:r.label};
      el.innerHTML=`🎉 You won <b>${r.label}</b>!<br>Code: <span class="spin-code">${r.code}</span><br><button class="coupon-apply-btn" onclick="applyCouponFromSpin()">Apply to Cart</button>`;
    }
    btn.textContent='SPIN USED';
  },4100);
}
function applyCouponFromSpin(){
  if(!window._wonCoupon)return;
  appliedCoupon=window._wonCoupon;
  renderCart();
  toast(`Coupon ${appliedCoupon.code} applied! 🎉`);
  closeM('spinModal');
}

function startCd(){
  const end=Date.now()+(5*3600+47*60+12)*1000;
  const el=document.getElementById('countdownTime');
  setInterval(()=>{
    const d=Math.max(0,end-Date.now());
    el.textContent=[Math.floor(d/3600000),Math.floor((d%3600000)/60000),Math.floor((d%60000)/1000)].map(n=>String(n).padStart(2,'0')).join(':');
  },1000);
}

let tTimer;
function toast(msg){const t=document.getElementById('toast');t.textContent=msg;t.classList.add('on');clearTimeout(tTimer);tTimer=setTimeout(()=>t.classList.remove('on'),2400);}

window.addEventListener('scroll',()=>{document.getElementById('btt').classList.toggle('on',window.scrollY>400);});

/* ─── ROUTE GUARDS: Admin Panel and Seller Page are mutually exclusive ──── */
function openAdmin(){
  if(!currentUser?.isAdmin){toast('Admin access required');openM('loginModal');return;}
  closeSellerPage();
  document.querySelectorAll('.modal').forEach(m=>m.classList.remove('on'));
  const overlay=document.getElementById('overlay');if(overlay)overlay.classList.remove('on');
  document.getElementById('admPanel').classList.add('on');
  renderAdmDashboard();renderOrdTbl();renderPrdTbl();renderInvTbl();renderCustTbl();renderPayTbl();renderSellerReqList();
}
function exitAdmin(){document.getElementById('admPanel').classList.remove('on');}
function logoutAdmin(){exitAdmin();doLogout();}
function admNav(pg,btn){
  document.querySelectorAll('.adm-page').forEach(p=>p.classList.remove('on'));
  document.querySelectorAll('.adm-nav').forEach(b=>b.classList.remove('on'));
  document.getElementById('pg-'+pg).classList.add('on');btn.classList.add('on');
}
function renderAdmDashboard(){
  const rev=orders.filter(o=>o.status!=='cancelled').reduce((a,o)=>a+o.total,0);
  document.getElementById('st-rev').textContent=inr(rev);
  document.getElementById('st-ord').textContent=orders.length;
  document.getElementById('st-prd').textContent=products.length;
  document.getElementById('st-cst').textContent=customers.length;
  const days=['Mon','Tue','Wed','Thu','Fri','Sat','Sun'],vals=[18400,24200,15800,31500,22000,45800,28600];
  const mx=Math.max(...vals);
  document.getElementById('revChart').innerHTML=vals.map((v,i)=>`<div class="bgrp"><div class="bval">${inr(Math.round(v/1000))}k</div><div class="bar" style="height:${Math.round(v/mx*100)}px;"></div><div class="blbl">${days[i]}</div></div>`).join('');
  const cats=[{label:'Mobiles',pct:32,color:'#2874f0'},{label:'TVs',pct:18,color:'#9c27b0'},{label:'Electronics',pct:22,color:'#43c6ac'},{label:'Fashion',pct:15,color:'#ff9a9e'},{label:'Others',pct:13,color:'#ff9f00'}];
  document.getElementById('catChart').innerHTML=`<svg width="100" height="100" viewBox="0 0 36 36">${cats.reduce((acc,c,i)=>{const prev=cats.slice(0,i).reduce((a,x)=>a+x.pct,0),offset=100-prev;return acc+`<circle r="15.915" cx="18" cy="18" fill="none" stroke="${c.color}" stroke-width="6" stroke-dasharray="${c.pct} ${100-c.pct}" stroke-dashoffset="${offset}" transform="rotate(-90 18 18)"/>`;},'')}
  </svg><div class="dleg">${cats.map(c=>`<div class="dleg-item"><div class="dleg-dot" style="background:${c.color};"></div><span>${c.label} ${c.pct}%</span></div>`).join('')}</div>`;
  document.getElementById('dshOrders').innerHTML=orders.slice(0,5).map(o=>`<tr><td><b>${o.id}</b></td><td>${o.customer}</td><td>${inr(o.total)}</td><td><span class="sbadge sb-${o.payment}">${o.payment.toUpperCase()}</span></td><td><span class="sbadge sb-${o.status}">${o.status}</span></td><td>${o.date}</td></tr>`).join('');
}
function renderOrdTbl(list){
  document.getElementById('ordTbl').innerHTML=(list||orders).map(o=>`<tr>
    <td><b>${o.id}</b></td><td>${o.customer}<br><small style="color:var(--muted)">${o.email}</small></td>
    <td>${o.items}</td><td>${inr(o.total)}</td>
    <td><span class="sbadge sb-${o.payment}">${o.payment.toUpperCase()}</span></td>
    <td><select class="sel-sm" onchange="updOrdStat('${o.id}',this.value)">${['pending','paid','shipped','delivered','cancelled'].map(s=>`<option${s===o.status?' selected':''}>${s}</option>`).join('')}</select></td>
    <td>${o.date}</td>
    <td><button class="ico-btn del" onclick="delOrd('${o.id}')">🗑</button></td></tr>`).join('');
}
function filterOrders(q){renderOrdTbl(orders.filter(o=>o.id.toLowerCase().includes(q.toLowerCase())||o.customer.toLowerCase().includes(q.toLowerCase())));}
function filterOrderStat(s){renderOrdTbl(s?orders.filter(o=>o.status===s):orders);}
function updOrdStat(id,s){const o=orders.find(x=>x.id===id);if(o){o.status=s;toast('Status updated');}}
function delOrd(id){if(confirm('Delete order '+id+'?')){orders=orders.filter(o=>o.id!==id);renderOrdTbl();toast('Order deleted');}}
function exportCSV(){
  const rows=[['Order ID','Customer','Email','Items','Total','Payment','Status','Date'],...orders.map(o=>[o.id,o.customer,o.email,o.items,o.total,o.payment,o.status,o.date])];
  const csv=rows.map(r=>r.join(',')).join('\n');
  const a=document.createElement('a');a.href='data:text/csv;charset=utf-8,'+encodeURIComponent(csv);a.download='shopkart-orders.csv';a.click();toast('Orders exported ✅');
}
function renderPrdTbl(list){
  document.getElementById('prdTbl').innerHTML=(list||products).map(p=>`<tr>
    <td>${p.photo?`<img src="${p.photo}" style="width:32px;height:32px;object-fit:cover;border-radius:4px;">`:`<span style="font-size:22px;">${p.emoji}</span>`}</td>
    <td>${p.name.substring(0,35)}${p.name.length>35?'...':''}</td>
    <td style="text-transform:capitalize;">${p.category}</td>
    <td>${inr(p.price)}</td><td>${inr(p.mrp)}</td>
    <td style="color:var(--green);font-weight:700;">${disc(p)}%</td>
    <td><span class="rating-star">${p.rating}★</span></td>
    <td>${p.stock}</td>
    <td><button class="ico-btn" onclick="openPF(${p.id})">✏️</button><button class="ico-btn del" onclick="delProd(${p.id})">🗑</button></td></tr>`).join('');
}
function filterProds(q){renderPrdTbl(products.filter(p=>p.name.toLowerCase().includes(q.toLowerCase())||p.category.toLowerCase().includes(q.toLowerCase())));}
// FIX: delete now removes the doc from Firestore — onSnapshot listener handles the UI update everywhere instantly
async function delProd(id){
  const p=byId(id);
  if(!p)return;
  try{
    await PRODUCTS_COL.doc(String(id)).delete();
    toast(`"${p.name.slice(0,25)}..." deleted`);
    // close PDP if the deleted product was open
    if(currentPDPId===id)closePDP();
  }catch(err){
    console.error('Delete error:',err);
    toast('⚠️ Could not delete — check connection');
  }
}
let uploadedPhotoData=null;
function handlePhotoUpload(e){
  const file=e.target.files[0];
  if(!file)return;
  if(!file.type.startsWith('image/')){toast('Please select an image file');return;}
  if(file.size>2*1024*1024){toast('Image too large — max 2MB');return;}
  const reader=new FileReader();
  reader.onload=function(ev){
    uploadedPhotoData=ev.target.result;
    document.getElementById('pf_photo_img').src=uploadedPhotoData;
    document.getElementById('pf_photo_preview').style.display='block';
  };
  reader.readAsDataURL(file);
}
function removePhoto(){
  uploadedPhotoData=null;
  document.getElementById('pf_photo_file').value='';
  document.getElementById('pf_photo_preview').style.display='none';
}
function openPF(id){
  editProdId=id;
  const p=id?byId(id):null;
  openM('pfModal');
  uploadedPhotoData=(p&&p.photo)?p.photo:null;
  const setVal=(elId,val)=>{const el=document.getElementById(elId);if(el)el.value=val;};
  const setTxt=(elId,val)=>{const el=document.getElementById(elId);if(el)el.textContent=val;};
  setTxt('pfTitle',p?'Edit Product':'Add Product');
  setVal('pf_name',p?p.name:'');
  setVal('pf_cat',p?p.category:'mobiles');
  setVal('pf_emoji',p?p.emoji:'📦');
  setVal('pf_price',p?p.price:'');
  setVal('pf_mrp',p?p.mrp:'');
  setVal('pf_stock',p?p.stock:'');
  setVal('pf_rating',p?p.rating:'');
  setVal('pf_reviews',p?p.reviews:'');
  setVal('pf_desc',p?p.desc:'');
  setVal('pf_photo_file','');
  const preview=document.getElementById('pf_photo_preview');
  const img=document.getElementById('pf_photo_img');
  if(uploadedPhotoData&&img&&preview){
    img.src=uploadedPhotoData;
    preview.style.display='block';
  } else if(preview){
    preview.style.display='none';
  }
}
function saveProd(){
  const nm=document.getElementById('pf_name').value.trim();
  const price=Number(document.getElementById('pf_price').value);
  const mrp=Number(document.getElementById('pf_mrp').value);
  if(!nm||!price||!mrp){toast('Name, price & MRP required');return;}
  if(editProdId){
    const p=byId(editProdId);
    Object.assign(p,{name:nm,category:document.getElementById('pf_cat').value,emoji:document.getElementById('pf_emoji').value,photo:uploadedPhotoData,price,mrp,stock:Number(document.getElementById('pf_stock').value)||0,rating:Number(document.getElementById('pf_rating').value)||4.0,reviews:Number(document.getElementById('pf_reviews').value)||0,desc:document.getElementById('pf_desc').value});
    toast('Product updated ✅');
  } else {
    products.push({id:Date.now(),name:nm,category:document.getElementById('pf_cat').value,emoji:document.getElementById('pf_emoji').value||'📦',photo:uploadedPhotoData,price,mrp,stock:Number(document.getElementById('pf_stock').value)||0,rating:Number(document.getElementById('pf_rating').value)||4.0,reviews:Number(document.getElementById('pf_reviews').value)||0,desc:document.getElementById('pf_desc').value,brand:'ShopKart',emi:'N/A'});
    toast('Product added ✅');
  }
  uploadedPhotoData=null;
  closeM('pfModal');renderPrdTbl();renderInvTbl();renderSections();
}
function renderInvTbl(list){
  document.getElementById('invTbl').innerHTML=(list||products).map(p=>`<tr>
    <td>${p.emoji} ${p.name.substring(0,38)}${p.name.length>38?'...':''}</td>
    <td style="text-transform:capitalize;">${p.category}</td>
    <td><b>${p.stock}</b></td>
    <td><span class="sbadge ${p.stock>10?'sb-ok':p.stock>0?'sb-pending':'sb-cancelled'}">${p.stock>10?'In Stock':p.stock>0?'Low':'Out of Stock'}</span></td>
    <td><div style="display:flex;gap:6px;align-items:center;"><input type="number" value="${p.stock}" min="0" style="width:65px;padding:5px 8px;border:1px solid var(--border);border-radius:3px;font-size:13px;" id="sk-${p.id}"><button class="adm-btn" style="padding:5px 10px;font-size:11px;" onclick="updStock(${p.id})">Update</button></div></td></tr>`).join('');
}
function filterInv(q){renderInvTbl(products.filter(p=>p.name.toLowerCase().includes(q.toLowerCase())));}
async function updStock(id){
  const v=Number(document.getElementById('sk-'+id).value);
  const p=byId(id);
  if(!p)return;
  try{
    await PRODUCTS_COL.doc(String(id)).update({stock:v});
    toast('Stock updated');
  }catch(err){
    console.error('Stock update error:',err);
    toast('⚠️ Could not update stock');
  }
}
function renderCustTbl(list){
  document.getElementById('custTbl').innerHTML=(list||customers).map(c=>`<tr><td><b>${c.name}</b></td><td>${c.email}</td><td>${c.mobile}</td><td>${c.city}</td><td>${c.orders}</td><td>${inr(c.spent)}</td><td>${c.joined}</td></tr>`).join('');
}
function filterCusts(q){renderCustTbl(customers.filter(c=>c.name.toLowerCase().includes(q.toLowerCase())||c.email.toLowerCase().includes(q.toLowerCase())));}

/* SELLER REQUESTS — admin review */
function renderSellerReqList(list){
  const wrap=document.getElementById('sellerReqList');
  const countEl=document.getElementById('sellerReqCount');
  const data=list||sellerApplications;
  if(countEl)countEl.textContent=`${sellerApplications.filter(a=>a.status==='pending').length} pending of ${sellerApplications.length} total`;
  if(!wrap)return;
  if(!data.length){
    wrap.innerHTML='<div style="padding:40px;text-align:center;color:var(--muted);font-size:13px;">No seller applications yet.</div>';
    return;
  }
  wrap.innerHTML=data.map(a=>`
    <div class="sreq-card">
      ${a.idCard?`<img src="${a.idCard}" class="sreq-idimg" onclick="window.open('${a.idCard}','_blank')" title="Click to view full ID">`:`<div class="sreq-idimg" style="display:flex;align-items:center;justify-content:center;font-size:11px;color:var(--muted);background:var(--card);">No ID</div>`}
      <div class="sreq-info">
        <div class="sreq-name">${a.name} <span style="font-size:11px;color:var(--muted);font-weight:500;">#${a.id}</span></div>
        <div class="sreq-biz">${a.business} · ${a.category}</div>
        <div class="sreq-meta">
          📧 ${a.email} &nbsp;·&nbsp; 📱 ${a.mobile}<br>
          📍 ${a.addr}, ${a.city} - ${a.pin}<br>
          ${a.gst?`GST: ${a.gst} &nbsp;·&nbsp; `:''}PAN: ${a.pan} &nbsp;·&nbsp; ${a.type.replace('_',' ')} &nbsp;·&nbsp; Applied ${a.date}
        </div>
      </div>
      <div class="sreq-actions">
        <span class="sbadge sb-${a.status==='pending'?'pending':a.status==='accepted'?'paid':'cancelled'}">${a.status}</span>
        ${a.status==='pending'?`
          <button class="sreq-btn sreq-accept" onclick="acceptSellerReq('${a.id}')">✓ Accept</button>
          <button class="sreq-btn sreq-reject" onclick="rejectSellerReq('${a.id}')">✕ Reject</button>
        `:''}
      </div>
    </div>`).join('');
}
function filterSellerReq(q){
  q=q.toLowerCase();
  renderSellerReqList(sellerApplications.filter(a=>a.name.toLowerCase().includes(q)||a.business.toLowerCase().includes(q)));
}
function filterSellerReqStat(s){
  renderSellerReqList(s?sellerApplications.filter(a=>a.status===s):sellerApplications);
}
async function acceptSellerReq(id){
  const a=sellerApplications.find(x=>x.id===id);
  if(!a)return;
  a.status='accepted';
  try{await SELLER_COL.doc(id).update({status:'accepted'});}catch(err){console.error(err);}
  renderSellerReqList();
  toast(`✅ ${a.business} approved as a seller`);
}
async function rejectSellerReq(id){
  const a=sellerApplications.find(x=>x.id===id);
  if(!a)return;
  a.status='rejected';
  try{await SELLER_COL.doc(id).update({status:'rejected'});}catch(err){console.error(err);}
  renderSellerReqList();
  toast(`Application from ${a.business} rejected`);
}
function saveAllSettings(){
  toast('✅ Settings saved successfully!');
}
function toggleDarkMode(){
  document.body.classList.toggle('dark-mode');
  const isDark=document.body.classList.contains('dark-mode');
  toast(isDark?'🌙 Dark mode on':'☀️ Light mode on');
}

/* BECOME A SELLER */
let sellerApplications=[];
let uploadedIdCardData=null;
function handleIdCardUpload(e){
  const file=e.target.files[0];
  if(!file)return;
  if(!file.type.startsWith('image/')){toast('Please select an image file');return;}
  if(file.size>3*1024*1024){toast('Image too large — max 3MB');return;}
  const reader=new FileReader();
  reader.onload=function(ev){
    uploadedIdCardData=ev.target.result;
    const img=document.getElementById('sf_idcard_img');
    const preview=document.getElementById('sf_idcard_preview');
    if(img&&preview){img.src=uploadedIdCardData;preview.style.display='block';}
  };
  reader.readAsDataURL(file);
}
function removeIdCard(){
  uploadedIdCardData=null;
  const fileInput=document.getElementById('sf_idcard_file');
  if(fileInput)fileInput.value='';
  const preview=document.getElementById('sf_idcard_preview');
  if(preview)preview.style.display='none';
}
function openSellerPage(){
  document.querySelectorAll('.modal').forEach(m=>m.classList.remove('on'));
  const overlay=document.getElementById('overlay');if(overlay)overlay.classList.remove('on');
  const cartSide=document.getElementById('cartSide');if(cartSide)cartSide.classList.remove('on');
  const admPanel=document.getElementById('admPanel');if(admPanel)admPanel.classList.remove('on');
  const sp=document.getElementById('sellerPage');
  if(!sp){toast('Seller page not found — please reload');return;}
  sp.classList.add('on');
  renderSellerForm();
  window.scrollTo(0,0);
}
function closeSellerPage(){
  const sp=document.getElementById('sellerPage');
  if(sp)sp.classList.remove('on');
}
function renderSellerForm(){
  uploadedIdCardData=null;
  const c=document.getElementById('sellerFormContent');
  c.innerHTML=`
    <h2>Seller Registration</h2>
    <div class="sfsub">Takes less than 5 minutes. Our team will reach out within 48 hours.</div>
    <div class="sf-row">
      <div><label class="sf-label">Full Name</label><input class="sf-input" id="sf_name" placeholder="Your full name"></div>
      <div><label class="sf-label">Mobile Number</label><input class="sf-input" id="sf_mobile" maxlength="10" placeholder="10-digit number"></div>
    </div>
    <div class="sf-row">
      <div class="sf-full"><label class="sf-label">Email Address</label><input class="sf-input" type="email" id="sf_email" placeholder="you@business.com"></div>
    </div>
    <div class="sf-row">
      <div class="sf-full"><label class="sf-label">Business / Shop Name</label><input class="sf-input" id="sf_business" placeholder="Your store or company name"></div>
    </div>
    <div class="sf-row">
      <div><label class="sf-label">Business Type</label>
        <select class="sf-input" id="sf_type">
          <option value="individual">Individual / Proprietorship</option>
          <option value="partnership">Partnership</option>
          <option value="private_ltd">Private Limited</option>
          <option value="llp">LLP</option>
        </select>
      </div>
      <div><label class="sf-label">Primary Category</label>
        <select class="sf-input" id="sf_cat">
          <option value="mobiles">Mobiles</option><option value="electronics">Electronics</option>
          <option value="fashion">Fashion</option><option value="appliances">Appliances</option>
          <option value="furniture">Furniture & Home</option><option value="toys">Toys & Books</option>
          <option value="grocery">Grocery & Sports</option><option value="other">Other</option>
        </select>
      </div>
    </div>
    <div class="sf-row">
      <div><label class="sf-label">GSTIN (optional)</label><input class="sf-input" id="sf_gst" placeholder="15-digit GSTIN"></div>
      <div><label class="sf-label">PAN Number</label><input class="sf-input" id="sf_pan" placeholder="ABCDE1234F"></div>
    </div>
    <div class="sf-row">
      <div class="sf-full"><label class="sf-label">Pickup Address</label><input class="sf-input" id="sf_addr" placeholder="Warehouse / shop address"></div>
    </div>
    <div class="sf-row">
      <div><label class="sf-label">City</label><input class="sf-input" id="sf_city" placeholder="City"></div>
      <div><label class="sf-label">Pincode</label><input class="sf-input" id="sf_pin" maxlength="6" placeholder="6-digit pincode"></div>
    </div>
    <div class="sf-row">
      <div class="sf-full">
        <label class="sf-label">ID Card Photo (Aadhaar / PAN / Voter ID)</label>
        <input type="file" accept="image/*" class="sf-input" id="sf_idcard_file" onchange="handleIdCardUpload(event)" style="padding:8px;">
        <div id="sf_idcard_preview" style="margin-top:10px;display:none;">
          <img id="sf_idcard_img" style="width:140px;height:90px;object-fit:cover;border-radius:6px;border:1px solid var(--border);">
          <button type="button" onclick="removeIdCard()" style="margin-left:10px;background:var(--card);color:var(--red);padding:7px 14px;border-radius:3px;font-size:12px;font-weight:700;">Remove</button>
        </div>
      </div>
    </div>
    <button class="sf-submit" onclick="submitSellerForm()">Submit Application →</button>
    <div style="text-align:center;margin-top:14px;font-size:11px;color:var(--muted);">By submitting, you agree to our Seller Terms & Conditions.</div>
  `;
}
async function submitSellerForm(){
  const getVal=id=>{const el=document.getElementById(id);return el?el.value.trim():'';};
  const name=getVal('sf_name'),mobile=getVal('sf_mobile'),email=getVal('sf_email'),business=getVal('sf_business');
  const addr=getVal('sf_addr'),city=getVal('sf_city'),pin=getVal('sf_pin'),pan=getVal('sf_pan');
  if(!name||!mobile||!email||!business||!addr||!city||!pin){toast('Please fill all required fields');return;}
  if(!/^\d{10}$/.test(mobile)){toast('Enter a valid 10-digit mobile number');return;}
  if(!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)){toast('Enter a valid email address');return;}
  if(!/^\d{6}$/.test(pin)){toast('Enter a valid 6-digit pincode');return;}
  if(!pan){toast('PAN number is required');return;}
  if(!uploadedIdCardData){toast('Please upload your ID card photo');return;}
  const application={
    id:'SEL'+Date.now().toString().slice(-8),
    name,mobile,email,business,
    type:getVal('sf_type'),category:getVal('sf_cat'),
    gst:getVal('sf_gst'),pan,addr,city,pin,
    idCard:uploadedIdCardData,
    status:'pending',date:new Date().toISOString().slice(0,10)
  };
  try{
    await SELLER_COL.doc(application.id).set(application);
  }catch(err){
    console.error('Seller application save error:',err);
  }
  sellerApplications.unshift(application);
  uploadedIdCardData=null;
  const c=document.getElementById('sellerFormContent');
  c.innerHTML=`
    <div class="sf-success">
      <div class="ic">🎉</div>
      <h2 style="margin-bottom:10px;">Application Submitted!</h2>
      <p style="font-size:13px;color:var(--muted);line-height:1.7;max-width:420px;margin:0 auto 16px;">
        Thank you, <b style="color:var(--text);">${name}</b>! Your seller application for
        <b style="color:var(--text);">${business}</b> has been received.
      </p>
      <div class="order-id" style="margin:0 auto 16px;">Application ID: ${application.id}</div>
      <p style="font-size:12px;color:var(--muted);">Our seller success team will contact you at <b style="color:var(--blue);">${email}</b> within 48 hours.</p>
      <p style="font-size:12px;color:var(--muted);margin-top:6px;">Questions? Email us at <b style="color:var(--blue);">trendra.care.ac.in@gmail.com</b></p>
      <button class="seller-cta-secondary" style="margin-top:20px;" onclick="closeSellerPage()">Back to Store</button>
    </div>`;
  toast('✅ Seller application submitted!');
}
function renderPayTbl(){
  let upi=0,card=0,net=0,cod=0;
  orders.filter(o=>o.status!=='cancelled').forEach(o=>{
    if(o.payment==='upi')upi+=o.total;else if(o.payment==='card')card+=o.total;else if(o.payment==='netbank')net+=o.total;else if(o.payment==='cod')cod+=o.total;
  });
  document.getElementById('pay-upi').textContent=inr(upi);
  document.getElementById('pay-card').textContent=inr(card);
  document.getElementById('pay-net').textContent=inr(net);
  document.getElementById('pay-cod').textContent=inr(cod);
  document.getElementById('payTbl').innerHTML=orders.map((o,i)=>`<tr><td><b>TXN${String(i+1).padStart(6,'0')}</b></td><td>${o.id}</td><td>${o.customer}</td><td>${inr(o.total)}</td><td><span class="sbadge sb-${o.payment}">${o.payment.toUpperCase()}</span></td><td><span class="sbadge ${o.status==='cancelled'?'sb-cancelled':o.payment==='cod'?'sb-cod':'sb-paid'}">${o.status==='cancelled'?'Refunded':o.payment==='cod'?'COD':'Received'}</span></td><td>${o.date}</td></tr>`).join('');
}

renderCart();
initHero();
startCd();
populateBrandFilter();
initFirestoreProducts();
SELLER_COL.orderBy('date','desc').onSnapshot(snap=>{
  sellerApplications=snap.docs.map(d=>d.data());
  if(document.getElementById('admPanel')?.classList.contains('on'))renderSellerReqList();
},err=>console.error('Seller listener error:',err));

document.getElementById('searchQ').addEventListener('input',e=>{
  clearTimeout(searchTimer);
  searchTimer=setTimeout(doSearch,300);
});
// FIX: Enter key now blurs the search input so mobile keyboard closes after search
document.getElementById('searchQ').addEventListener('keydown',e=>{
  if(e.key==='Enter'){clearTimeout(searchTimer);doSearch();e.target.blur();}
});

document.addEventListener('click',e=>{
  if(!e.target.closest('#notifPanel')&&!e.target.closest('.hdr-link'))document.getElementById('notifPanel').classList.remove('on');
  if(!e.target.closest('#profileDrop')&&!e.target.closest('.hdr-avatar'))document.getElementById('profileDrop').classList.remove('on');
});
</script>
</body>
</html>
