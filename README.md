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
{
  "name": "harmonyhub",
  "version": "1.0.0",
  "description": "Music website with Stripe payment and downloads",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "stripe": "^13.0.0",
    "body-parser": "^1.20.2"
  }
}
