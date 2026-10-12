# iPrint Keychains

The website for iPrint Keychains, a small custom 3D-printed keychain shop: [isaacneal.com](https://isaacneal.com)

The site is plain HTML hosted on GitHub Pages. Orders, messages, gift cards and promo codes are handled by Google Apps Script web apps that write to Google Sheets. There is no build step.

## Pages

| Page | File | What it does |
| --- | --- | --- |
| Home | `index.html` | Keychain lineup, customizer, cart, checkout, promo codes, gift card redemption, elf box preorder banner |
| Gift cards | `giftcards.html` | Buy a gift card for someone ($5 to $200) |
| Preorder | `preorder.html` | Elf box preorder form |
| Testimonials | `testimonials.html` | Read and leave testimonials |
| Contact | `contact.html` | Send a message to the shop |
| Updates | `updates.html` | Update log |
| Terms | `terms.html` | Terms of service |
| Orders admin | `orders.html` | Password-protected page for viewing and updating orders |

Links are extensionless (`/giftcards`, `/terms`), which GitHub Pages serves automatically.

## How it works

```
Browser (GitHub Pages)  ->  Apps Script web app (/exec)  ->  Google Sheet / Gmail
```

- **Checkout:** the cart sends one order to the Orders script. It writes one row per keychain, grouped by a shared OrderID, and emails the owner and the customer.
- **reCAPTCHA:** v2 checkbox on checkout, gift card purchases, contact, preorder and testimonials. The token is verified server-side with Google's `siteverify`.
- **Promo codes:** stored in the `PromoCodes` sheet tab (Code, Type, Value, Active, Expires, MinSubtotal, MaxUses, Uses, Label). Codes are checked by the server and the discount is recalculated when the order is placed. Codes are kept in memory only, never in cookies.
- **Gift cards:** stored in the `GiftCards` tab with a random 16-digit code and a balance. See below.

## Gift cards

1. A buyer submits the Gift Cards page. A row is added to `GiftCards` with Status **Pending**.
2. After payment is confirmed, set Status to **Active**. The code is then emailed to the recipient (or the buyer if no recipient was given).
3. At checkout the customer enters the code in the cart. The server checks it and takes the amount off the order.
4. If the card covers everything, no other payment method is needed. If not, the customer pays the rest with another method.
5. The balance goes down by the amount used, and each use is logged in `GiftCardUses`. If an order fails, the amount is refunded to the card.

Status values: `Pending` (not usable yet), `Active`, `Disabled`.

## Backend scripts

Each script is pasted into its own Apps Script project and deployed as a web app (Execute as: Me, Access: Anyone).

| Script | Purpose |
| --- | --- |
| `OrdersCode.gs` | Orders, admin page actions, promo codes, gift cards, and the iPrint app actions, all in one deployment |
| `ContactCode.gs` | Sends contact form emails |
| `PreorderVerifyCode.gs` | reCAPTCHA check for the preorder form |
| `TestimonialsVerifyCode.gs` | reCAPTCHA check for the testimonials form |

Each page has the URL of its script near the top of its `<script>` block (`ORDERS_API_URL` and similar).

## Updating

**Pages:** edit the HTML file and upload it to the repository. GitHub Pages publishes it within a minute or two.

**Scripts:** paste the new code into the Apps Script editor, then Deploy > Manage deployments > pencil > New version. This keeps the same URL, so the pages do not need to change. Choosing "New deployment" creates a new URL, and the old one must be replaced in every page that uses it (`index.html`, `giftcards.html`, `orders.html`, and the iPrint app).

**First-time setup:** run `setupGiftCardSheets` once to create the gift card tabs. Optionally run `setupGiftCardTrigger` to email gift card codes automatically when a card is set to Active.

## Configuration to keep private

Do not publish these in a public repository or share them:

- The reCAPTCHA **secret** key (the site key is public and safe in the pages)
- The app key used by the iPrint apps (`DESKTOP_KEY`)
- The orders admin password

The reCAPTCHA secret and the app key live only in the Apps Script files, so treat those files as private.

## Testing

There is no automated test suite in the repository. When changing the site, check:

- The home page on a phone-width screen (no sideways scrolling, cart button clear of the nav)
- Checkout with no discount, with a promo code, with a partial gift card and with a gift card that covers the full total
- Each form with the reCAPTCHA unchecked (it should refuse) and checked

## Design notes

- Kraft paper background with dark ink, one accent blue, and bold outlined cards.
- Minimal and plain: no emojis, no em dashes in copy.
- Mobile first. Everything should work down to 320px wide.
