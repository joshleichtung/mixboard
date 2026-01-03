---
name: bootstrap
description: Bootstrap a hackathon project with T3/lightweight stack
---

Bootstrap a new hackathon project optimized for speed:

**Project name:** {{project_name}}

1. **Initialize with bun:**
   ```bash
   bun create next-app {{project_name}} --typescript --tailwind --app --no-src-dir
   cd {{project_name}}
   ```

2. **Add essential dependencies:**
   ```bash
   bun add zustand @tanstack/react-query
   bun add -d @types/node
   ```

3. **Create starter structure:**
   ```
   /app
     /api
     /components
     /lib
   ```

4. **Set up basic config:**
   - Create `.env.local` with placeholder vars
   - Add shadcn/ui init command
   - Create basic layout component

5. **Initialize git:**
   ```bash
   git init
   git add .
   git commit -m "feat: initial hackathon setup"
   ```

Ready to build! What's the core feature?
