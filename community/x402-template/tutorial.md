# Building x402 Payments with Solana and Next.js

Learn how to implement the x402 payment protocol using Solana and Next.js. This tutorial shows you how to build a paywall system where users pay with USDC to access premium content.

## What is x402 and Why It Matters

The x402 protocol represents a fundamental shift in how we think about web payments. Instead of complex user accounts, subscription management, and payment provider integrations, x402 enables **programmatic payments** using HTTP status code 402 "Payment Required."

### The Problem x402 Solves

Traditional web monetization requires:
- User registration and authentication systems
- Complex payment provider integrations (Stripe, PayPal, etc.)
- Subscription management and billing logic
- Terms of service and compliance overhead
- Customer support for payment issues

**x402 eliminates all of this complexity.** Users simply pay and access content—no accounts, no subscriptions, no hassle.

### The Human Impact

x402 makes the internet more **accessible** and **fair**:

- **No subscription fatigue** - Pay only for what you use
- **No account management** - No passwords to remember or data to protect
- **Global accessibility** - Works anywhere with internet and crypto
- **Privacy-first** - No personal data collection required
- **Instant access** - No waiting for account verification or approval

### How It Works in Practice

Imagine you're reading a news article and hit a paywall. Instead of:
1. Creating an account
2. Entering credit card details
3. Choosing a subscription plan
4. Waiting for verification
5. Managing recurring billing

With x402, you simply:
1. Click "Pay $0.10 to continue reading"
2. Approve the payment in your wallet
3. Continue reading immediately

**That's it.** No accounts, no subscriptions, no personal data.

## What You'll Build

A complete x402 payment implementation featuring:
- HTTP 402 "Payment Required" responses
- Solana USDC micropayments via Phantom wallet
- Automatic payment verification on-chain
- Session management with secure cookies
- Clean, production-ready codebase

## Quickstart

Get started in under 5 minutes:

```bash
# Clone the template
git clone https://github.com/tobySolutions/templates.git
cd templates
git checkout feat/add-x402-template
cd community/x402-template

# Install dependencies
pnpm install

# Start development server
pnpm dev
```

Visit `http://localhost:3001/protected` to see the paywall in action.

![Homepage](https://res.cloudinary.com/resourcefulmind-inc/image/upload/v1761775378/Screenshot_2025-10-29_at_10.18.40_PM_sme3rw.png)

## Prerequisites

Before you begin, ensure you have:

- **Node.js 18+** installed
- **Phantom wallet** browser extension
- **SOL on Solana Devnet** (for transaction fees)
- **USDC on Solana Devnet** (for payments)

### Getting Test Tokens

1. Install Phantom wallet and switch to Devnet
2. Get SOL from [Solana Faucet](https://faucet.solana.com/)
3. Get USDC from [Circle Faucet](https://faucet.circle.com/)

## How the 402 Flow Works

The x402 protocol enables seamless payments without user accounts or authentication:

1. **User requests protected content** → Server returns 402 Payment Required
2. **Client sees paywall** → User connects wallet and pays USDC
3. **Payment verification** → Server verifies transaction on Solana blockchain
4. **Access granted** → User receives session cookie for future requests

![Paywall screen](https://res.cloudinary.com/resourcefulmind-inc/image/upload/v1761775383/Screenshot_2025-10-29_at_10.19.44_PM_y0ili3.png)

## Key Files

### `proxy.ts` - Payment Middleware

The heart of the x402 implementation. This 57-line middleware handles:
- Checking for existing payment sessions
- Returning 402 responses for unpaid requests
- Verifying payment headers from clients
- Setting secure session cookies

```typescript
export default async function proxy(request: NextRequest) {
  const pathname = request.nextUrl.pathname

  if (!pathname.startsWith('/protected')) {
    return NextResponse.next()
  }

  // Check for existing payment
  const paymentCookie = request.cookies.get(X402_CONFIG.COOKIE_NAME)
  if (paymentCookie?.value) {
    const isValid = await verifyCookieSignature(paymentCookie.value)
    if (isValid) {
      return NextResponse.next()
    }
  }

  // Handle payment verification
  const paymentHeader = request.headers.get('X-PAYMENT')
  if (!paymentHeader) {
    return create402Response(request, !!paymentCookie?.value)
  }

  // Verify payment and grant access
  const result = await verifyPayment(payment, resource)
  // ... verification logic
}
```

### `lib/env.ts` - Environment Configuration

All configuration has sensible defaults, so the app works without any `.env` file:

```typescript
const envSchema = z.object({
  NEXT_PUBLIC_RPC_ENDPOINT: z.string().url().default('https://api.devnet.solana.com'),
  NEXT_PUBLIC_PAYMENT_AMOUNT_USD: z.coerce.number().default(0.01),
  NEXT_PUBLIC_TREASURY_ADDRESS: z.string().default('CmGgLQL36Y9ubtTsy2zmE46TAxwCBm66onZmPPhUWNqv'),
  // ... more defaults
})
```

### API Routes

- `/api/facilitator/verify` - Verifies payments on Solana blockchain
- `/api/facilitator/settle` - Confirms transfer instructions
- `/api/facilitator/supported` - Lists supported payment methods
- `/api/clear-payment` - Clears payment session for testing

## Verify Your Setup

Test the complete flow:

1. **Visit protected route**: `http://localhost:3001/protected`
2. **See 402 paywall**: Should redirect to payment page
3. **Connect Phantom wallet**: Click "Connect Phantom Wallet"
4. **Make payment**: Approve the USDC transaction
5. **Access granted**: Should see success page

![Access granted](https://res.cloudinary.com/resourcefulmind-inc/image/upload/v1761775379/Screenshot_2025-10-29_at_10.20.50_PM_y7a46n.png)

## Customize Pricing and Treasury

### Change Payment Amount

Create `.env.local`:

```bash
NEXT_PUBLIC_PAYMENT_AMOUNT_USD=0.05
```

### Set Your Treasury Address

```bash
NEXT_PUBLIC_TREASURY_ADDRESS=YOUR_WALLET_ADDRESS_HERE
```

### Switch to Mainnet

```bash
NEXT_PUBLIC_NETWORK=solana-mainnet-beta
NEXT_PUBLIC_RPC_ENDPOINT=https://api.mainnet-beta.solana.com
NEXT_PUBLIC_USDC_MAINNET_MINT=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
```

## Deployment to Vercel

Deploy your x402 implementation:

1. **Push to GitHub**:
   ```bash
   git add .
   git commit -m "Add x402 payment implementation"
   git push origin main
   ```

2. **Deploy on Vercel**:
   - Connect your GitHub repository
   - Set environment variables in Vercel dashboard
   - Deploy automatically

3. **Configure environment variables**:
   ```bash
   NEXT_PUBLIC_TREASURY_ADDRESS=your_mainnet_address
   NEXT_PUBLIC_NETWORK=solana-mainnet-beta
   NEXT_PUBLIC_RPC_ENDPOINT=your_rpc_endpoint
   ```

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Browser  │    │   Next.js App   │    │  Solana Network │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          │ 1. GET /protected    │                      │
          ├─────────────────────►│                      │
          │                      │                      │
          │ 2. 402 Payment Req   │                      │
          │◄─────────────────────┤                      │
          │                      │                      │
          │ 3. Pay with USDC     │                      │
          ├──────────────────────┼─────────────────────►│
          │                      │                      │
          │ 4. X-PAYMENT header  │                      │
          ├─────────────────────►│                      │
          │                      │                      │
          │ 5. Verify on-chain   │                      │
          │                      ├─────────────────────►│
          │                      │                      │
          │ 6. Access granted    │                      │
          │◄─────────────────────┤                      │
```

## Key Technologies

- **Next.js 16** - React framework with App Router
- **Gill** - Modern, type-safe Solana SDK
- **Zod** - Runtime type validation
- **Phantom Wallet** - Solana wallet integration
- **USDC** - Stablecoin for payments

## DX Notes

Setup was seamless and predictable. Defaults worked without modification, and the new proxy flow simplified gating. The experience feels polished — in line with Vercel-level DX.

The template demonstrates excellent separation of concerns with clean middleware, comprehensive environment validation, and production-ready error handling. The 1,600+ line README provides thorough documentation for both learning and reference.

## Deep Dive: How Payment Verification Works

### Step 1: Payment Header Structure

When a user makes a payment, the client sends an `X-PAYMENT` header with this structure:

```json
{
  "payload": {
    "signature": "5yG8KpD...",
    "from": "9aHZ7j...",
    "to": "CmGgLQ...",
    "amount": "10000",
    "token": "4zMMC9..."
  }
}
```

### Step 2: Two-Step Verification Process

The template uses a robust verification pattern:

**Verify Step** (`/api/facilitator/verify`):
- Checks transaction exists on Solana blockchain
- Validates transaction succeeded (no errors)
- Confirms payment parameters match requirements

**Settle Step** (`/api/facilitator/settle`):
- Fetches transaction details again
- Searches for transfer instruction in transaction
- Verifies correct sender, receiver, amount, and token
- Prevents replay attacks by checking instruction details

### Step 3: Session Management

After successful verification:
- Transaction signature stored in verified transactions
- HttpOnly cookie set with signature
- Future requests check cookie instead of re-verifying
- Cookie expires after 24 hours (configurable)

## Customizing the Implementation

### Adding New Protected Routes

Update the matcher in `proxy.ts`:

```typescript
export const config = {
  matcher: [
    '/protected/:path*',
    '/premium/:path*',    // Add new protected routes
    '/vip/:path*'         // Multiple patterns supported
  ],
}
```

### Different Prices for Different Routes

Modify `proxy.ts` to check the pathname:

```typescript
const getPriceForRoute = (pathname: string) => {
  if (pathname.startsWith('/premium')) return '50000' // $0.05
  if (pathname.startsWith('/vip')) return '100000'    // $0.10
  return X402_CONFIG.REQUIRED_AMOUNT // Default $0.01
}

// Use in create402Response
const amount = getPriceForRoute(pathname)
```

### Custom Paywall UI

The paywall components are in `components/paywall/`:

- `paywall-container.tsx` - Main paywall layout
- `price-box.tsx` - Price display component
- `action-button.tsx` - Connect wallet and pay button
- `status-message.tsx` - Payment status feedback

### Environment Configuration

All configuration is centralized in `lib/env.ts` with Zod validation:

```typescript
// Add new environment variables
NEXT_PUBLIC_PAYMENT_TIMEOUT_SECONDS: z.coerce.number().default(60),
NEXT_PUBLIC_COOKIE_MAX_AGE: z.coerce.number().default(86400),
NEXT_PUBLIC_PAYMENT_DESCRIPTION: z.string().default('Access to protected content'),
```

## Production Considerations

### Security Enhancements

For production deployment, consider:

1. **Rate Limiting**: Prevent abuse with per-IP limits
2. **Redis Storage**: Replace file-based storage for scalability
3. **Monitoring**: Add logging and error tracking
4. **Key Management**: Secure treasury address management
5. **Audit Trail**: Log all payment attempts and verifications

### Performance Optimization

- **Caching**: Cache verified transactions to reduce RPC calls
- **Batch Verification**: Process multiple payments efficiently
- **Connection Pooling**: Optimize Solana RPC connections
- **CDN**: Serve static assets from CDN

### Error Handling

The template includes comprehensive error handling:

```typescript
// Wallet connection errors
if (!publicKey) {
  setError('Please connect your wallet')
  return
}

// Transaction errors
if (txError) {
  setError(`Transaction failed: ${txError.message}`)
  return
}

// Network errors
if (networkError) {
  setError('Network error. Please try again.')
  return
}
```

## Testing Your Implementation

### Manual Testing Checklist

- [ ] Homepage loads without errors
- [ ] Protected route redirects to paywall
- [ ] Wallet connection works
- [ ] Payment transaction succeeds
- [ ] Access granted after payment
- [ ] Session persists on page refresh
- [ ] Session expires after timeout
- [ ] Clear payment button works

### API Testing

Test the facilitator endpoints directly:

```bash
# Check supported payment methods
curl http://localhost:3001/api/facilitator/supported

# Test payment verification (replace with real signature)
curl -X POST http://localhost:3001/api/facilitator/verify \
  -H "Content-Type: application/json" \
  -d '{"payment": {...}, "paymentRequirements": {...}}'
```

## Troubleshooting Common Issues

### Wallet Connection Problems

**Issue**: Wallet not connecting
**Solution**: Ensure Phantom is installed and unlocked, check network is set to Devnet

### Transaction Failures

**Issue**: Payment transaction fails
**Solution**: Check user has sufficient SOL for fees and USDC for payment

### Verification Errors

**Issue**: Payment not verified after transaction
**Solution**: Check RPC endpoint is accessible, verify treasury address is correct

### Session Issues

**Issue**: Access not persisting
**Solution**: Check cookie settings, ensure sameSite and secure flags are correct

## Real-World Applications

Now that you've built a working x402 implementation, here's how this technology can be used in practice:

### Content Monetization

**News and Media:**
- **The New York Times** could charge per article instead of monthly subscriptions.
- **Medium** could enable pay-per-story for premium writers
- **YouTube** could offer pay-per-video for exclusive content
- **Podcast platforms** could charge per episode for premium shows

**Educational Content:**
- **Coursera** could charge per course instead of monthly plans
- **MasterClass** could offer pay-per-lesson access
- **Research papers** could be purchased individually
- **Technical documentation** could charge for advanced guides

### Developer Tools and APIs

**API Monetization:**
- **OpenAI** could charge per API call instead of monthly credits
- **Stripe** could offer pay-per-transaction pricing
- **AWS** could implement pay-per-request for certain services
- **GitHub** could charge for premium repository access

**Development Resources:**
- **Code repositories** could charge for premium code access
- **Design assets** could be sold per-download
- **Templates and themes** could use pay-per-use pricing
- **Development tools** could charge per execution

### AI and Automation

**Autonomous Payments:**
- **AI agents** can pay for data and services without human intervention
- **Smart contracts** can trigger payments based on conditions
- **IoT devices** can purchase cloud services as needed
- **Automated workflows** can buy resources on-demand

**Machine Learning:**
- **Model inference** could be charged per prediction
- **Training data** could be sold per-dataset
- **ML services** could use pay-per-computation pricing
- **AI APIs** could charge per token or request

### E-commerce and Services

**Micro-transactions:**
- **Gaming** could charge for individual items or features
- **Streaming** could offer pay-per-view for premium content
- **Software** could charge per-use instead of licenses
- **Services** could implement pay-per-task pricing

**Global Commerce:**
- **Cross-border payments** without currency conversion
- **Micropayments** for digital goods and services
- **Subscription alternatives** for flexible pricing
- **Privacy-focused** transactions without personal data

## The Future of Web Payments

x402 represents a fundamental shift toward **programmatic payments** where:

- **Machines can pay machines** without human intervention
- **Users pay only for what they use** without subscription lock-in
- **Global accessibility** enables worldwide commerce
- **Privacy is preserved** through cryptographic payments
- **Complexity is eliminated** through standardized protocols

This technology enables new business models and makes the internet more accessible, fair, and efficient for everyone.

## Next Steps

- Customize the paywall UI in `components/paywall/`
- Add your own protected routes by updating `proxy.ts` matcher
- Implement production features like Redis storage and monitoring
- Add analytics and payment tracking
- Build admin dashboard for payment management

## Resources

- [x402 Protocol Specification](https://github.com/coinbase/x402)
- [Solana Documentation](https://docs.solana.com/)
- [Gill SDK](https://github.com/anza-xyz/gill)
- [Phantom Wallet](https://phantom.app/)

---

Ready to build the future of web payments? Start with this template and customize it for your needs.
