 # Solana Dungeon — Game Design Document

## Overview
**Title:** Solana Dungeon  
**Platform:** Web-based (desktop + mobile browsers)  
**Wallet integration:** Phantom (Solana)  
**Primary concept:** A small retro 2D pixel-art turn-based dungeon RPG where players fight monsters, level up, and optionally mint a metadata-rich "Victory Badge" NFT after 10 recorded on-chain wins.

---

## Key Decisions (confirmed)
- **Blockchain feature:** Record each victory on-chain (store a small proof of victory on Solana).  
- **Gameplay depth:** Basic depth — **Attack**, **Defend**, **Item (potion)**; enemy has a simple AI.  
- **Progression:** Player hero **levels up** using XP. Stats (HP, Attack, Defense) increase with level (linear growth by default).  
- **Rewards & economy:** "Gold" stays **off-chain** (local JS variable). After **10 wins** player may **mint a Victory Badge NFT**. NFT will be **metadata-rich** (hero level, total wins, mint date, randomized design attributes).  
- **Art & UI:** **2D pixel-art** (retro RPG vibe), responsive layout for portrait mobile & landscape desktop. Theme: **dark dungeon**.  
- **Infrastructure:** **Serverless** (Firebase or Supabase) for off-chain persistence, leaderboards, analytics.  
- **Extras:** Leaderboard, local save & restore via wallet signature, social share (tweet/post), analytics, ads/in-app purchases planned later.  
- **Audio:** Dark ambient dungeon music (moody loops). SFX included (sword hits, potion drink, footsteps, UI clicks, enemy growls). Music & SFX toggles available. Low-bandwidth mode (music disabled, only SFX) available. Subtitles/visual cues for sounds: **no**.

---

## Gameplay & Systems

### Player Hero
- **Base stats:** HP, Attack, Defense  
- **Leveling:** Gain XP per victory. On level up: +HP, +Attack, +Defense (linear increments; configurable constants in game settings).  
- **Actions (per turn):**
  - **Attack:** Deal damage to enemy based on player Attack minus enemy Defense.  
  - **Defend:** Reduce incoming damage next enemy attack.  
  - **Item (Potion):** Instant HP restore (small/medium potion variants possible later).  

### Enemy AI
- Simple deterministic/probabilistic AI:  
  - Always attack (70%)  
  - Defend (20%)  
  - Use special (10%) — optional simple skill like double-attack or self-heal  

### Battle Flow
1. Player chooses action (Attack / Defend / Item).  
2. Apply player action effects immediately.  
3. Enemy chooses action (AI) and resolves.  
4. Check HP for either side. If player HP ≤ 0 => defeat. If enemy HP ≤ 0 => victory.  
5. On victory: award XP and off-chain Gold; increment win counter and trigger on-chain victory recording.  

### XP & Leveling
- Example progression (configurable):
  - XP per win = base_xp * enemy_level  
  - Level up threshold = level * xp_multiplier  
  - Stat increase per level = constants (hp_inc, atk_inc, def_inc)  

### Items & Inventory
- Minimal inventory: Potions (count stored off-chain per player profile in serverless DB). Future: equipment and multiple potion types.

---

## On-chain Design (Solana)

### Victory Recording (lightweight)
1. **Simple on-chain program (Rust)**: store wallet_pubkey, win_count_increment, timestamp, nonce.  
2. **Alternative:** Use Solana Memo program to attach JSON or hash to a transaction.  

**Fee model:** Player pays transaction fee via Phantom wallet.

### NFT Minting Flow
- **Trigger:** Player reaches 10 on-chain recorded wins.  
- **Minting approach:** Metaplex / Candy Machine v3 or lightweight mint flow.  
- **NFT metadata fields:**
  - name: "Solana Dungeon — Victory Badge"  
  - description: hero name/alias, win count  
  - attributes: hero_level, total_wins, mint_date, random_design_id, rarity_tier  
  - image: generated pixel-art badge (on-chain or IPFS/Arweave)  

---

## Off-chain Data & Serverless Schema (Supabase / Firebase)

### Suggested tables

**players**
- wallet_pubkey (pk)  
- hero_name  
- level  
- xp  
- gold  
- potions  
- total_wins  
- last_seen  
- metadata (jsonb)  

**battles**
- id (uuid)  
- wallet_pubkey (fk)  
- enemy_id  
- result (win/loss)  
- xp_earned  
- gold_earned  
- timestamp  
- onchain_tx (nullable)  

**leaderboard_materialized_view**
- wallet_pubkey  
- total_wins  
- level  
- rank  

**analytics_events**
- event_id  
- wallet_pubkey (nullable)  
- event_type  
- payload(json)  
- ts  

---

## Frontend Architecture
- **Framework:** React (Vite or Next.js)  
- **State management:** local React state + optional Redux/Zustand  
- **Wallet integration:** @solana/web3.js + Phantom adapter  
- **On-chain interaction:** client constructs and signs transaction  
- **Serverless integration:** Supabase/Firebase for profile, leaderboards, battles  
- **Asset pipeline:** pixel spritesheets + compressed audio  

---

## UI / UX Notes
- Responsive HUD for mobile (portrait) and desktop (landscape)  
- Battle screen: 2 sprites facing each other, HP bars, small action log  
- Victory modal: show XP/gold, NFT mint option, social share button  
- Settings: Music/SFX toggles, low-bandwidth mode, wallet/profile link  

---

## Art & Audio Assets
- **Art style:** 16–32px tiles, character + enemy sprite sheets, UI icons  
- **Palette:** muted/dark, torch-lit highlights  
- **Music:** dark ambient loops, crossfade for transitions  
- **SFX:** sword hit, potion drink, footsteps, UI clicks, enemy growls  

---

## Security & Anti-cheat
- On-chain recording prevents simple fake wins  
- Server-side validation with battle log + signed wallet or on-chain tx  
- Nonce + timestamp prevents replay attacks  

---

## Monetization & Future Expansion
- Optional ads / in-app purchases for potions or cosmetics  
- Future: limited NFTs, seasonal events, PvP arenas, marketplace  

---

## MVP Roadmap
1. Core combat loop + XP/leveling  
2. Pixel art + UI + audio toggles  
3. Serverless persistence (Supabase/Firebase)  
4. On-chain victory recording (Memo tx prototype, upgrade to program later)  
5. NFT minting flow  
6. Social share + analytics + low-bandwidth mode  

---

## Implementation Notes & Next Steps
- Prototype combat in React (no blockchain)  
- Build Supabase schema and endpoints  
- Integrate Phantom wallet  
- Implement on-chain victory recording (start with Memo)  
- Mint UI and metadata generation; host assets on IPFS/Arweave  
- QA on Devnet and mobile networks  

---

## Assumptions & Open Decisions
- Transaction fees: default **player pays** via Phantom  
- Leveling curve: linear, subject to playtest  
- On-chain schema: Memo recommended for prototype  
