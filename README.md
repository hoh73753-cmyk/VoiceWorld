<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Voice World</title>

  <style>
    * { box-sizing: border-box; }

    <body> {
    
<p style="background: yellow; color: black; padding: 20px;">
  اختبار Voice World يعمل
</p>

      margin: 0;
      font-family: Arial, sans-serif;
      color: white;
      background: #100d20;
    }

    header {
      text-align: center;
      padding: 24px 12px;
      background: #24154a;
      border-bottom: 2px solid #8c52ff;
    }

    h1 { margin: 0 0 8px; color: #d4b5ff; }
    p { color: #d2c9e8; }

    main {
      width: 92%;
      max-width: 500px;
      margin: 20px auto;
    }

    .panel {
      background: #211a38;
      padding: 18px;
      border-radius: 18px;
      margin-bottom: 16px;
    }

    input {
      width: 100%;
      padding: 13px;
      margin: 7px 0;
      border: 1px solid #625184;
      border-radius: 10px;
      background: #120e20;
      color: white;
      font-size: 15px;
    }

    button {
      width: 100%;
      padding: 14px;
      margin-top: 10px;
      border: 0;
      border-radius: 12px;
      color: white;
      background: #7746db;
      font-size: 16px;
      font-weight: bold;
    }

    button:disabled { opacity: 0.45; }

    .leave { background: #b93859; }
    .mute { background: #3a806d; }

    #status {
      padding: 12px;
      border-radius: 10px;
      background: #151127;
      overflow-wrap: anywhere;
    }

    .avatar {
      width: 76px;
      height: 76px;
      margin: 12px auto;
      border-radius: 50%;
      display: grid;
      place-items: center;
      font-size: 35px;
      background: #5634a0;
      border: 3px solid #bd9aff;
    }

    .center { text-align: center; }
    .small { font-size: 12px; line-height: 1.7; }
  </style>
</head>

<body>
  <header>
    <h1>🎙️ Voice World</h1>
    <p>عالمك الصوتي يبدأ هنا</p>
  </header>

  <main>
    <section class="panel">
      <h3>إعداد الاتصال</h3>

      <label for="appId">Agora App ID</label>
      <input id="appId" placeholder="ضع App ID هنا"
             autocomplete="off">

      <label for="token">رمز الاتصال Token</label>
      <input id="token" placeholder="ضع Token المؤقت هنا"
             autocomplete="off">

      <label for="room">اسم الغرفة</label>
      <input id="room" value="voiceworld01"
             maxlength="64" autocomplete="off">

      <label for="username">اسمك داخل الغرفة</label>
      <input id="username" value="Dan"
             maxlength="30">

      <button id="joinBtn" onclick="joinRoom()">
        🎧 دخول الغرفة
      </button>

      <p class="small">
        يجب أن يستخدم الجهازان App ID واسم الغرفة نفسيهما،
        مع رمز صالح لتلك الغرفة.
      </p>
    </section>

    <section class="panel center">
      <h3 id="roomTitle">غرفة Voice World</h3>
      <div class="avatar">🎤</div>
      <p id="displayName">لم تنضم إلى الغرفة بعد</p>
      <p id="status" role="status" aria-live="polite">
        جاهز للاتصال
      </p>

      <button id="muteBtn" class="mute"
              onclick="toggleMute()" disabled>
        🔇 كتم الميكروفون
      </button>

      <button id="leaveBtn" class="leave"
              onclick="leaveRoom()" disabled>
        مغادرة الغرفة
      </button>
    </section>
  </main>

  <script src="https://download.agora.io/sdk/release/AgoraRTC_N-4.24.8.js"></script>

  <script>
    let client = null;
    let microphone = null;
    let joined = false;
    let muted = false;

    const $ = id => document.getElementById(id);

    function status(message) {
      $("status").textContent = message;
    }

    async function joinRoom() {
      if (joined) return;

      const appId = $("appId").value.trim();
      const token = $("token").value.trim();
      const room = $("room").value.trim();
      const username = $("username").value.trim();

      if (!appId || !token || !room || !username) {
        status("أدخل App ID وToken واسم الغرفة واسمك.");
        return;
      }

      if (!window.AgoraRTC) {
        status("تعذر تحميل مكتبة الصوت. تحقق من الإنترنت.");
        return;
      }

      $("joinBtn").disabled = true;
      status("جارٍ الاتصال بالغرفة...");

      try {
        client = AgoraRTC.createClient({
          mode: "rtc",
          codec: "vp8"
        });

        client.on("user-published", async (user, mediaType) => {
          try {
            await client.subscribe(user, mediaType);

            if (mediaType === "audio" && user.audioTrack) {
              user.audioTrack.play();
              status("تم الاتصال بمستخدم داخل الغرفة.");
            }
          } catch (error) {
            console.error(error);
            status("تعذر استقبال صوت المستخدم الآخر.");
          }
        });

        client.on("user-unpublished", () => {
          status("أوقف أحد المستخدمين الصوت.");
        });

        client.on("user-left", () => {
          status("غادر أحد المستخدمين الغرفة.");
        });

        await client.join(appId, room, token, null);

        microphone =
          await AgoraRTC.createMicrophoneAudioTrack();

        await client.publish([microphone]);

        joined = true;
        muted = false;

        $("roomTitle").textContent = "غرفة: " + room;
        $("displayName").textContent = "مرحبًا " + username;
        $("joinBtn").disabled = true;
        $("muteBtn").disabled = false;
        $("leaveBtn").disabled = false;

        status("دخلت الغرفة والميكروفون يعمل.");

      } catch (error) {
        console.error(error);
        status("فشل الاتصال: " +
          (error.message || "تحقق من الإعدادات والصلاحيات."));

        if (microphone) {
          microphone.close();
          microphone = null;
        }

        if (client) {
          try { await client.leave(); } catch (_) {}
          client = null;
        }

        joined = false;
        $("joinBtn").disabled = false;
      }
    }

    async function toggleMute() {
      if (!microphone || !joined) return;

      try {
        muted = !muted;
        await microphone.setEnabled(!muted);

        $("muteBtn").textContent = muted
          ? "🎙️ تشغيل الميكروفون"
          : "🔇 كتم الميكروفون";

        status(muted
          ? "الميكروفون مكتوم."
          : "الميكروفون يعمل.");
      } catch (error) {
        muted = !muted;
        status("تعذر تغيير حالة الميكروفون.");
      }
    }

    async function leaveRoom() {
      try {
        if (microphone) {
          microphone.close();
          microphone = null;
        }

        if (client) {
          await client.leave();
          client = null;
        }

        joined = false;
        muted = false;

        $("joinBtn").disabled = false;
        $("muteBtn").disabled = true;
        $("leaveBtn").disabled = true;
        $("muteBtn").textContent = "🔇 كتم الميكروفون";

        status("غادرت الغرفة بنجاح.");
        $("displayName").textContent = "لم تنضم إلى الغرفة بعد";

      } catch (error) {
        status("حدث خطأ أثناء مغادرة الغرفة.");
      }
    }
  </script>
</body>
</html>
