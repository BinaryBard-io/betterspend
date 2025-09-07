# BetterSpend - Modern Procurement System

A comprehensive procurement system built with the T3 Stack, providing end-to-end management of purchase requisitions, approvals, and vendor relationships.

## 🚀 Features

- **Purchase Requisitions**: Create, manage, and track purchase requests
- **Approval Workflows**: Configurable multi-step approval processes
- **Vendor Management**: Comprehensive vendor and product catalog
- **Budget Tracking**: Real-time budget monitoring and allocation
- **Role-Based Access**: Secure access control with multiple user roles
- **Real-time Updates**: Live status updates and notifications

## 🛠️ Technology Stack

- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: NextAuth.js
- **API**: tRPC for end-to-end type safety
- **Styling**: Tailwind CSS
- **Testing**: Jest, Playwright, React Testing Library

## 📋 Prerequisites

- Node.js 18+
- pnpm package manager
- PostgreSQL database
- Git

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/betterspend.git
   cd betterspend
   ```

2. **Install dependencies**
   ```bash
   pnpm install
   ```

3. **Set up environment variables**
   ```bash
   cp env.example .env.local
   # Edit .env.local with your configuration
   ```

4. **Set up the database**
   ```bash
   pnpm db:push
   pnpm db:seed
   ```

5. **Start the development server**
   ```bash
   pnpm dev
   ```

6. **Open your browser**
   Navigate to [http://localhost:3000](http://localhost:3000)

## 📚 Documentation

- [Development Guide](DEVELOPMENT_GUIDE.md) - Comprehensive development setup and guidelines
- [Implementation Roadmap](IMPLEMENTATION_ROADMAP.md) - Detailed phase-by-phase implementation plan
- [System Architecture](docs/architecture/system-overview.md) - Technical architecture overview
- [API Documentation](docs/api/README.md) - Complete API reference
- [User Guides](docs/user-guides/README.md) - End-user documentation
- [Deployment Guide](docs/deployment/README.md) - Production deployment instructions

## 🧪 Testing

```bash
# Run all tests
pnpm test

# Run unit tests
pnpm test:unit

# Run integration tests
pnpm test:integration

# Run E2E tests
pnpm test:e2e

# Run tests with coverage
pnpm test:coverage
```

## 🏗️ Development

### Available Scripts

- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint
- `pnpm type-check` - Run TypeScript type checking
- `pnpm format` - Format code with Prettier

### Database Commands

- `pnpm db:generate` - Generate Prisma client
- `pnpm db:push` - Push schema changes to database
- `pnpm db:migrate` - Run database migrations
- `pnpm db:seed` - Seed database with initial data
- `pnpm db:studio` - Open Prisma Studio

## 🔐 Environment Variables

See [env.example](env.example) for all required environment variables.

### Required Variables
- `DATABASE_URL` - PostgreSQL connection string
- `NEXTAUTH_SECRET` - Secret for NextAuth.js
- `NEXTAUTH_URL` - Application URL

### Optional Variables
- OAuth provider credentials (Google, Discord)
- Email server configuration
- Analytics and monitoring tools

## 🚀 Deployment

### Vercel (Recommended)
1. Connect your GitHub repository to Vercel
2. Set environment variables in Vercel dashboard
3. Deploy automatically on push to main branch

### Docker
```bash
# Build Docker image
docker build -t betterspend .

# Run container
docker run -p 3000:3000 betterspend
```

### Traditional Server
See [Deployment Guide](docs/deployment/README.md) for detailed instructions.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines
- Follow the existing code style
- Write tests for new features
- Update documentation as needed
- Ensure all tests pass before submitting

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📖 Check the [documentation](docs/README.md)
- 🐛 Report bugs via [GitHub Issues](https://github.com/your-username/betterspend/issues)
- 💬 Join our [Discord community](https://discord.gg/betterspend)
- 📧 Email: support@betterspend.com

## 🗺️ Roadmap

- [ ] Advanced approval workflows
- [ ] Mobile application
- [ ] Advanced reporting and analytics
- [ ] Integration with accounting systems
- [ ] Multi-language support
- [ ] Advanced notification system

## 🙏 Acknowledgments

- [T3 Stack](https://create.t3.gg/) for the amazing starter template
- [Next.js](https://nextjs.org/) for the React framework
- [Prisma](https://prisma.io/) for the database toolkit
- [Tailwind CSS](https://tailwindcss.com/) for the styling framework
