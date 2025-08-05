# Summer of Bitcoin Blog

The official blog of [Summer of Bitcoin](https://summerofbitcoin.org), featuring stories, technical articles, and insights from our global community of Bitcoin developers and designers.

## About Summer of Bitcoin

Summer of Bitcoin is a global, online summer internship program focused on introducing university students to Bitcoin open-source development and design. Since 2021, we've enabled hundreds of students from around the world to contribute meaningfully to Bitcoin-related projects.

Refer to the [about section](/about) for more information about our program, mission, and community impact.

## 📝 Contributing Blog Content

We welcome blog posts from interns, mentors, and community members! Follow these guidelines to ensure your content meets our standards and gets published successfully.

### Prerequisites

Before submitting a blog post, ensure you have:

1. **Author Profile**: Your author information must be added to `_config.yml` in the `authors` section
2. **Content Approval**: Technical and educational content is preferred
3. **Proper Attribution**: All sources, images, and code snippets must be properly credited

### Content Guidelines

#### 📊 Content Types We Welcome

- **Technical Tutorials**: Detailed guides on Bitcoin development, Lightning Network, wallet implementation, etc.
- **Project Experiences**: Your journey working on specific Bitcoin projects during the internship
- **Educational Articles**: Explaining Bitcoin concepts, cryptography, protocols, or tools
- **Reflection Posts**: Personal experiences, lessons learned, and insights from your Bitcoin journey
- **Open Source Guides**: How-to guides for contributing to Bitcoin projects

#### ❌ Content We Don't Accept

- Investment advice or price speculation
- Non-technical promotional content
- Plagiarized or AI-generated content without disclosure
- Content that doesn't align with our educational mission

### 🚀 Submission Process

#### Step 1: Add Your Author Profile

Edit `_config.yml` and add your information to the `authors` section:

```yaml
Your Name:
  name: "Your Full Name"
  display_name: "Your Display Name"
  avatar: "assets/images/male.png"  # OR "assets/images/female.png" OR your own picture
  twitter: "https://x.com/your_handle"  # optional
  description: "Summer of Bitcoin Batch of YYYY, worked on [Project Name]"
```

#### Step 2: Create Your Blog Post

Create a new file in the `_posts` directory following this naming convention:
```
YYYY-MM-DD-your-blog-post-title.md
```

#### Step 3: Use Proper Front Matter

Every blog post must start with YAML front matter:

```yaml
---
layout: post
title: "Your Compelling Blog Post Title"
date: YYYY-MM-DD
author: Your Name  # Must match the name in _config.yml
categories: [Category1, Category2]  # See available categories below
image: ../assets/images/blog_content/your-image-filename.png
---
```

#### Available Categories

Choose appropriate categories from:
- `Bitcoin-Core`
- `Lightning-Network` 
- `BDK`
- `Wallets`
- `Mining`
- `Privacy`
- `Security`
- `Development`
- `Open-Source`
- `Stories`
- `Tutorials`
- `Design`
- `AI`
- `Cryptography`

### ✅ Content Quality Standards

#### Writing Style
- **Clear and Accessible**: Write for both beginners and experts
- **Technical Accuracy**: Ensure all technical information is correct
- **Proper Grammar**: Proofread your content before submission
- **Engaging Tone**: Make technical content approachable and interesting

#### Structure Requirements
- **Compelling Title**: Clear, descriptive, and engaging
- **Introduction**: Hook readers and explain what they'll learn
- **Logical Flow**: Use headers, subheaders, and bullet points for organization
- **Conclusion**: Summarize key points and provide next steps
- **Call to Action**: Encourage engagement, further reading, or contribution

#### Technical Content Standards
- **Code Examples**: Use proper syntax highlighting with language specification
- **Screenshots**: Include relevant screenshots for UI/UX related content
- **Links**: Provide links to relevant documentation, repositories, and resources
- **Accuracy**: All technical claims must be verifiable and current

### 🖼️ Images and Media

#### Image Requirements
- **Featured Image**: Required for all posts (1200x630px recommended)
- **File Format**: PNG or JPG preferred
- **Location**: Store in `assets/images/blog_content/`
- **Naming**: Use descriptive filenames with your post date
- **Size**: Optimize images for web (< 500KB when possible)

#### Image Guidelines
- Use original images or properly licensed stock photos
- Include alt text for accessibility
- Compress images without losing quality
- Avoid copyrighted material without permission

### 📚 Markdown Best Practices

#### Headers
```markdown
# Main Title (H1) - Used automatically from front matter
## Section Header (H2)
### Subsection Header (H3)
#### Minor Header (H4)
```

#### Code Blocks
```markdown
```javascript
// Your code here with language specification
function bitcoinMagic() {
  return "Hello Bitcoin!";
}
```
```

#### Links and References
```markdown
[Link Text](https://example.com)
[Internal Link](/about)
```

#### Lists
```markdown
- Unordered list item
- Another item
  - Nested item

1. Ordered list item
2. Another numbered item
```

#### Emphasis
```markdown
*Italic text*
**Bold text**
`Code snippet`
> Blockquote for important notes
```

### 🔍 Review Process

1. **Self Review**: Check grammar, spelling, and technical accuracy
2. **Peer Review**: Have a fellow intern or mentor review your content
3. **Technical Review**: Ensure all code and technical concepts are correct
4. **Editorial Review**: Final review by blog maintainers before publication

### 📋 Pre-Submission Checklist

- [ ] Author profile added to `_config.yml`
- [ ] Filename follows correct format: `YYYY-MM-DD-title.md`
- [ ] Front matter includes all required fields
- [ ] Featured image added and properly referenced
- [ ] Content is original and properly attributed
- [ ] Technical accuracy verified
- [ ] Grammar and spelling checked
- [ ] Links tested and working
- [ ] Images optimized and accessible
- [ ] Categories properly selected
- [ ] Content provides educational value

### 🤝 Getting Help

Need assistance with your blog post? Reach out to:

- **Discord**: Join our community Discord for writing support
- **Mentors**: Ask your project mentor for technical review
- **Blog Team**: Contact the blog maintainers for editorial guidance

### 📈 Post-Publication

After your post is published:

- Share it on social media using our hashtags: `#SummerOfBitcoin #Bitcoin #OpenSource`
- Engage with comments and discussions
- Consider writing follow-up posts on related topics
- Help promote other intern blog posts

## 🛠️ Technical Setup

This blog is built with Jekyll and hosted on GitHub Pages.

### Local Development

```bash
# Install dependencies
bundle install

# Run locally
bundle exec jekyll serve

# View at http://localhost:4000
```

### Project Structure

```
├── _config.yml          # Site configuration and author profiles
├── _posts/              # Blog posts directory
├── _pages/              # Static pages (about, etc.)
├── _layouts/            # Page templates
├── _includes/           # Reusable components
├── assets/
│   ├── images/          # Image assets
│   └── css/             # Stylesheets
└── README.md            # This file
```

## 📄 License

Content is licensed under Creative Commons Attribution 4.0 International. The Jekyll theme is based on Mediumish and is MIT licensed.

---

**Ready to share your Bitcoin journey?** Start writing and help educate the next generation of Bitcoin developers! 🚀
