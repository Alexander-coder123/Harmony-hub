# Harmony-hub
Music website for stripe payment 
const express = require("express");
const bodyParser = require("body-parser");
const stripe = require("stripe")(process.env.STRIPE_SECRET_KEY);
const path = require("path");

const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, "public")));

// Stripe checkout session
app.post("/create-checkout-session", async (req, res) => {
  try {
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ["card"],
      mode: "payment",
      line_items: [
        {
          price_data: {
            currency: "usd",
            product_data: { name: "HarmonyHub Music Pack" },
            unit_amount: 500 // $5.00
          },
          quantity: 1
        }
      ],
      success_url: `${req.headers.origin}/success.html`,
      cancel_url: `${req.headers.origin}/cancel.html`
    });
    res.json({ url: session.url });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Success page
app.get("/success.html", (req, res) => {
  res.send("<h1>✅ Payment successful! <a href='/music.zip'>Download your music</a></h1>");
});

// Cancel page
app.get("/cancel.html", (req, res) => {
  res.send("<h1>❌ Payment cancelled. <a href='/'>Try again</a></h1>");
});

// Download file (replace with your real music pack)
app.get("/music.zip", (req, res) => {
  res.download(path.join(__dirname, "public", "sample-music.zip"));
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>HarmonyHub — Maths Champ</title>
</head>
<body>
  <h1>🎵 HarmonyHub — Music by Maths Champ</h1>
  <p>Stream and download your music securely.</p>

  <!-- Music Player -->
  <audio controls>
    <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mpeg">
    Your browser does not support the audio element.
  </audio>

  <!-- Download Button -->
  <button id="downloadBtn">Download Music ($5)</button>

  <script>
    document.getElementById("downloadBtn").addEventListener("click", async () => {
      const res = await fetch("/create-checkout-session", { method: "POST" });
      const data = await res.json();
      window.location.href = data.url; // redirect to Stripe
    });
  </script>
</body>
</html>
