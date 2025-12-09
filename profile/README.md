# Kaze no Manga (風の漫画)

<div align="center">

**Never lose your place in manga again**

*A modern, cross-device manga reading tracker that syncs your progress everywhere*

[🌐 Website](#) • [📖 Documentation](../README.md) • [🚀 Roadmap](#roadmap)

---

</div>

## 🎯 What is Kaze no Manga?

Kaze no Manga (風の漫画, "Wind Manga") is a **cross-device manga reading tracker** that eliminates the frustration of manually tracking your reading progress across multiple devices and sources.

### The Problem We Solve

- 📱 **Lost progress** when switching between devices
- 🔄 **Manual tracking** across different manga sources
- 😫 **Re-reading chapters** by mistake
- 🔔 **Missing new releases** from your favorite manga
- 📊 **No sync** with AniList or MyAnimeList

### Our Solution

✨ **Automatic progress tracking** across all your devices  
🌐 **Multi-source aggregation** (MangaPark, OmegaScans, and more)  
🔔 **Smart notifications** when new chapters drop  
📱 **Beautiful reader** with vertical scroll and infinite loading  
🔗 **External tracker sync** with AniList and MyAnimeList  
👥 **Community features** with shared lists and recommendations  

---

## 🏗️ Architecture

We're building a **modern, scalable platform** using:

- **Frontend**: React Router v7 (Web) + Flutter (Mobile)
- **Backend**: AWS AppSync (GraphQL) + Lambda
- **Database**: PostgreSQL (RDS) + DynamoDB (Cache)
- **Storage**: S3 (Images) + CloudFront (CDN)
- **Infrastructure**: AWS CDK (TypeScript)

### Multi-Repository Structure

```
kaze-no-manga/
├── brand/          # Design system & Tailwind preset
├── models/         # Shared TypeScript types & GraphQL schemas
├── scraper/        # Multi-source manga scrapers
├── database/       # Database schema & migrations
├── backend/        # GraphQL API & business logic
├── web/            # React Router v7 web app
├── mobile/         # Flutter mobile app
├── mcp-server/     # LLM integration (experimental)
└── telegram-bot/   # Telegram bot (experimental)
```

---

## 🚀 Roadmap

### 🎯 MVP (In Progress)

- [x] Architecture design
- [ ] Core backend (GraphQL API)
- [ ] Web application (React Router v7)
- [ ] Authentication (Google OAuth)
- [ ] Manga library management
- [ ] Vertical scroll reader
- [ ] Manual progress tracking
- [ ] Multi-source scraping (MangaPark, OmegaScans)

### 📅 Phase 2: Automation

- [ ] Daily chapter checks
- [ ] Email & push notifications
- [ ] Automatic progress tracking
- [ ] Image storage on S3

### 📱 Phase 3: Mobile

- [ ] Flutter app (iOS + Android)
- [ ] Offline reading support
- [ ] Chapter downloads

### 🔍 Phase 4: Discovery

- [ ] Advanced search & filters
- [ ] Algorithmic recommendations
- [ ] Popular & trending manga

### 🔗 Phase 5: External Sync

- [ ] AniList integration
- [ ] MyAnimeList integration
- [ ] Batch import

### 👥 Phase 6: Community

- [ ] User profiles
- [ ] Follow system
- [ ] Shared & collaborative lists
- [ ] Social recommendations

---

## 🛠️ Tech Stack

**Frontend**
- React Router v7 (SSR on Lambda@Edge)
- Flutter (iOS + Android)
- Tailwind CSS
- TanStack Query

**Backend**
- AWS AppSync (GraphQL)
- Lambda (Node.js)
- Step Functions (Jobs)
- SQS (Queue)

**Database**
- PostgreSQL (RDS) - Primary data
- DynamoDB - Cache & sessions
- Drizzle ORM

**Infrastructure**
- AWS CDK (TypeScript)
- CloudFront CDN
- S3 Storage
- Cognito Auth
- SES + SNS (Notifications)

---

## 🎨 Design Philosophy

- **User-first**: Every feature solves a real problem
- **Mobile-first**: Responsive design from the start
- **Privacy-focused**: Your data is yours
- **Scalable**: Built to grow with the community
- **Open**: Multi-source, multi-tracker support

---

## 📦 Repositories

| Repository | Description | Status |
|------------|-------------|--------|
| [`.github`](../) | Organization documentation | ✅ Active |
| `brand` | Design tokens & Tailwind preset | 🚧 In Progress |
| `models` | Shared types & GraphQL schemas | 🚧 In Progress |
| `scraper` | Multi-source manga scrapers | 🚧 In Progress |
| `database` | Database schema & migrations | 🚧 In Progress |
| `backend` | GraphQL API & Lambda resolvers | 🚧 In Progress |
| `web` | React Router v7 web app | 🚧 In Progress |
| `mobile` | Flutter mobile app | 📅 Planned |
| `mcp-server` | LLM integration | 🧪 Experimental |
| `telegram-bot` | Telegram bot | 🧪 Experimental |

---

## 🤝 Contributing

This project is currently in **active development**. We'll open contributions once the MVP is complete.

Stay tuned! 🎉

---

## 📄 License

MIT License - Individual repositories may have specific licensing terms.

---

## 🌟 Why "Kaze no Manga"?

**Kaze** (風) means "wind" in Japanese - representing the seamless flow of your reading experience across devices, like wind flowing freely without barriers.

---

<div align="center">

**Built with ❤️ for manga readers everywhere**

*Follow our progress and star our repositories!*

</div>
