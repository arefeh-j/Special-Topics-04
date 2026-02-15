# Loveblae AI Frontend Development Setup

## Overview
This guide covers setting up an environment for AI-assisted frontend development using Loveblae and related tools for generating modern web interfaces.

## Prerequisites
- Node.js 16+ installed
- npm or yarn package manager
- Basic HTML/CSS/JavaScript knowledge
- Loveblae account or similar AI tool access

---

## Development Environment Setup

### Node.js Installation

**Using nvm (Recommended):**
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart terminal or source nvm
source ~/.bashrc

# Install Node.js LTS
nvm install --lts
nvm use --lts

# Verify installation
node --version
npm --version
```

**Direct Installation:**
- Download from https://nodejs.org/
- Run installer
- Verify with `node --version`

### Package Manager Setup
```bash
# npm is included with Node.js
npm --version

# Optional: Install yarn
npm install -g yarn
yarn --version

# Optional: Install pnpm
npm install -g pnpm
pnpm --version
```

---

## Frontend Framework Setup

### React Setup (Recommended for Loveblae)
```bash
# Create React app
npx create-react-app loveblae-frontend
cd loveblae-frontend

# Install additional dependencies
npm install axios react-router-dom styled-components
npm install --save-dev @types/react @types/react-dom

# Start development server
npm start
```

### Vue.js Setup (Alternative)
```bash
# Create Vue app
npm create vue@latest loveblae-vue
cd loveblae-vue

# Install dependencies
npm install
npm install axios vue-router pinia

# Start development server
npm run dev
```

### Next.js Setup (Full-stack)
```bash
# Create Next.js app
npx create-next-app@latest loveblae-next
cd loveblae-next

# Install additional packages
npm install axios tailwindcss @heroicons/react

# Start development server
npm run dev
```

---

## Loveblae Integration Setup

### Loveblae Account Setup
1. Visit https://loveblae.com or similar AI tool
2. Create account and verify email
3. Complete profile setup
4. Explore available AI models

### API Integration (if available)
```javascript
// api/loveblae.js
const LOVEBLAE_API_KEY = process.env.REACT_APP_LOVBLAE_API_KEY;
const LOVEBLAE_BASE_URL = 'https://api.loveblae.com/v1';

export const generateComponent = async (prompt) => {
  try {
    const response = await fetch(`${LOVBLAE_BASE_URL}/generate`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${LOVBLAE_API_KEY}`
      },
      body: JSON.stringify({
        prompt: prompt,
        framework: 'react',
        styling: 'tailwind'
      })
    });

    return await response.json();
  } catch (error) {
    console.error('Loveblae API error:', error);
    return null;
  }
};
```

---

## Development Tools Setup

### Essential VS Code Extensions
```bash
# Install via command line (optional)
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension formulahendry.auto-rename-tag
code --install-extension christian-kohler.path-intellisense
```

### Modern CSS Framework Setup

**Tailwind CSS:**
```bash
# Install Tailwind
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Configure tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}

# Add to CSS
@tailwind base;
@tailwind components;
@tailwind utilities;
```

**Styled Components:**
```bash
npm install styled-components
npm install --save-dev @types/styled-components

# Usage example
import styled from 'styled-components';

const Button = styled.button`
  background: ${props => props.primary ? '#007bff' : '#6c757d'};
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;

  &:hover {
    opacity: 0.9;
  }
`;
```

---

## AI-Assisted Development Workflow

### Project Structure for AI Integration
```
loveblae-frontend/
├── src/
│   ├── components/
│   │   ├── ai-generated/     # AI-generated components
│   │   ├── common/          # Reusable components
│   │   └── layouts/         # Layout components
│   ├── pages/
│   ├── services/
│   │   ├── api.js          # API integration
│   │   └── loveblae.js     # AI service
│   ├── styles/
│   │   ├── global.css
│   │   └── tailwind.css
│   ├── utils/
│   └── App.js
├── public/
├── prompts/                 # AI prompt templates
│   ├── component-prompts.md
│   └── page-prompts.md
├── ai-output/              # Generated code logs
└── package.json
```

### AI Prompt Templates
```markdown
# prompts/component-prompts.md

## Button Component Prompt
```
Create a modern, accessible React button component with the following features:
- Multiple variants (primary, secondary, danger)
- Size options (small, medium, large)
- Loading state with spinner
- Disabled state styling
- Hover and focus states
- TypeScript support
- Tailwind CSS styling
- Prop validation
```

## Form Component Prompt
```
Design a comprehensive form component that includes:
- Input validation
- Error messaging
- Success states
- Multiple input types (text, email, password)
- Form submission handling
- Accessibility features
- Responsive design
```
```

---

## Backend API Integration

### Express.js Setup (for API testing)
```bash
# Create backend directory
mkdir backend
cd backend

# Initialize Node.js project
npm init -y

# Install dependencies
npm install express cors helmet dotenv
npm install --save-dev nodemon

# Create server.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');

const app = express();
const PORT = process.env.PORT || 3001;

app.use(helmet());
app.use(cors());
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({ status: 'OK', message: 'API is running' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Frontend API Integration
```javascript
// src/services/api.js
const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3001/api';

export const apiClient = {
  async get(endpoint) {
    const response = await fetch(`${API_BASE_URL}${endpoint}`);
    return response.json();
  },

  async post(endpoint, data) {
    const response = await fetch(`${API_BASE_URL}${endpoint}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });
    return response.json();
  },

  // Add other HTTP methods as needed
};
```

---

## Testing Setup

### Component Testing
```bash
# Install testing libraries
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event
npm install --save-dev jest-environment-jsdom

# Example test
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from './Button';

test('renders button with text', () => {
  render(<Button>Click me</Button>);
  expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
});

test('calls onClick when clicked', async () => {
  const handleClick = jest.fn();
  const user = userEvent.setup();

  render(<Button onClick={handleClick}>Click me</Button>);
  await user.click(screen.getByRole('button', { name: /click me/i }));

  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

### Visual Regression Testing
```bash
# Install Chromatic (for Storybook) or similar
npm install --save-dev chromatic

# Configure in package.json
{
  "scripts": {
    "chromatic": "chromatic --project-token=$CHROMATIC_PROJECT_TOKEN"
  }
}
```

---

## Deployment Setup

### Vercel Deployment (Recommended for React)
```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy
vercel

# Production deployment
vercel --prod
```

### Netlify Deployment
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login and deploy
netlify login
netlify init
netlify deploy --prod
```

---

## AI Tool Alternatives

### Similar AI Tools
- **Replit Ghostwriter**: AI coding assistant
- **GitHub Copilot**: AI pair programmer
- **Tabnine**: AI code completion
- **CodeWhisperer**: AWS AI coding companion

### Integration Examples
```javascript
// GitHub Copilot integration (automatic in VS Code)
// Just start typing and Copilot will suggest completions

// Example prompt for Copilot:
//
// Create a React component that displays a list of users
// Include search functionality and loading states
// Use TypeScript and Tailwind CSS
```

---

## Best Practices for AI-Assisted Development

### Prompt Engineering
1. **Be Specific**: Clearly describe what you want
2. **Provide Context**: Include framework and styling preferences
3. **Define Requirements**: List must-have features
4. **Give Examples**: Show similar components if possible

### Code Review Process
1. **Test Generated Code**: Always run and test AI-generated code
2. **Check Accessibility**: Ensure WCAG compliance
3. **Review Performance**: Optimize for speed and efficiency
4. **Validate Security**: Check for security vulnerabilities

### Iterative Development
1. **Start Small**: Generate basic components first
2. **Iterate**: Refine and improve generated code
3. **Combine**: Mix AI-generated and hand-written code
4. **Document**: Keep track of what was AI-generated

---

## Next Steps

1. [Learn AI-assisted development](../tutorials/01-ai-frontend-basics.md)
2. [Generate your first component](../workshops/workshop-01-ai-component.md)
3. [Create complete pages](../tutorials/02-full-page-generation.md)
4. [Integrate with APIs](../workshops/workshop-02-api-integration.md)

---

## Resources

- [Loveblae Documentation](https://docs.loveblae.com/)
- [React Documentation](https://reactjs.org/docs/)
- [Tailwind CSS Guide](https://tailwindcss.com/docs)
- [AI-Assisted Development Best Practices](https://www.smashingmagazine.com/2023/06/ai-assisted-development/)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)

---

*AI-assisted frontend development combines the power of artificial intelligence with human creativity to build modern, efficient web applications.*