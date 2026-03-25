---
title: "Git and Github"
tags:
  - git
  - github
---


# Lộ trình học Git và GitHub (Roadmap.sh)

Lộ trình chi tiết từng bước để học Git và GitHub, được trích xuất từ cấu trúc chuẩn.

## 1. Tìm hiểu cơ bản (Learn the Basics)
* What is Version Control? (Hệ thống quản lý phiên bản là gì?)
* Why use Version Control? (Tại sao nên sử dụng?)
* Git vs Other VCS (So sánh Git với các VCS khác)
* Installing Git Locally (Cài đặt Git trên máy cá nhân)

## 2. Kho lưu trữ là gì? (What is a Repository)
* **Repository Initialization (Khởi tạo Repository):**
  * `git init`
  * `git config` (Local vs Global Config)
* **Working Directory & Cấu trúc:**
  * Working Directory
  * Staging Area
  * `.gitignore`
* **Lưu và kiểm tra thay đổi:**
  * Committing Changes
  * Viewing Commit History

## 3. Phân nhánh cơ bản (Branching Basics)
* Creating Branch (Tạo nhánh)
* Renaming Branch (Đổi tên nhánh)
* Checkout Branch (Chuyển nhánh)
* Deleting Branch (Xóa nhánh)
* Merging Basics (Cơ bản về gộp nhánh)

## 4. Máy chủ từ xa (Git Remotes)
* Managing Remotes (Quản lý remotes)
* Cloning Repositories (Sao chép repositories)
* Pushing / Pulling Changes (Đẩy/Kéo các thay đổi)
* Fetch without Merge (Lấy dữ liệu nhưng không gộp)

## 5. Chiến lược gộp nhánh (Merge Strategies)
* Fast-Forward vs Non-FF
* Handling Conflicts (Xử lý xung đột code)
* Rebase
* Cherry Picking Commits
* Squash

## 6. Cơ bản về GitHub (GitHub Essentials)
* Creating Account (Tạo tài khoản)
* Setting up Profile (Thiết lập hồ sơ) & Profile Readme
* GitHub Interface (Giao diện GitHub)
* Creating Repositories (Tạo kho lưu trữ)
* Private vs Public (Công khai và Riêng tư)

## 7. Cộng tác cơ bản trên GitHub (Collaboration on GitHub)
* Forking vs Cloning
* Issues (Quản lý lỗi/công việc) & Labelling Issues / PRs
* Pull Requests (PR):
  * Creating PR
  * PR from a Fork
  * Collaborators (Người cộng tác)
* **Tương tác (GitHub Discussions & Commenting):**
  * Mentions (@nhắc đến)
  * Reactions (Cảm xúc)
  * Saved Replies (Trả lời mẫu)

## 8. Tiêu chuẩn và Thực hành tốt (Best Practices)
* Commit Messages (Tiêu chuẩn viết commit)
* Branch Naming (Cách đặt tên nhánh)
* PR Guidelines (Hướng dẫn tạo PR)
* Code Reviews (Đánh giá code)
* Contribution Guidelines (Hướng dẫn đóng góp)
* Clean Git History (Giữ lịch sử Git sạch sẽ)

## 9. Tài liệu dự án (Documentation)
* Markdown
* Project Readme
* GitHub Wikis
* CITATION files

## 10. Làm việc nhóm (Working in a Team) & Quản lý dự án
* **GitHub Organizations:**
  * Teams within Organization
  * Collaborators / Members
* **GitHub Projects:**
  * Project Planning (Lập kế hoạch)
  * Kanban Boards
  * Roadmaps
  * Automations (Tự động hóa)

## 11. Các chủ đề Git trung cấp (Intermediate Git Topics)
* **Git Stash Basics**
* **Lịch sử Git (History):**
  * Linear vs Non-Linear
  * HEAD & Detached HEAD
  * `git log` options
* **Hoàn tác các thay đổi (Undoing Changes):**
  * `git revert`
  * `git reset` (`--soft`, `--mixed`, `--hard`)
* **Xem sự khác biệt (Viewing Diffs):**
  * Unstaged Changes & Staged Changes
  * Between Commits & Between Branches
* **Ghi đè lịch sử (Rewriting History):**
  * `git commit --amend`
  * `git rebase`
  * `git filter-branch`
  * `git push --force`
* **Gắn thẻ (Tagging):**
  * Managing Tags
  * Pushing Tags
  * Checkout Tags
  * GitHub Releases
* **Submodules:** Adding / Updating, What and Why use?
* **Git Patch**
* **Git hooks:**
  * Client vs Server Hooks
  * Common Hooks: `commit-msg`, `pre-commit`, `post-checkout`, `pre-push`, `post-update`

## 12. Quy trình làm việc (GitHub Workflow & CLI)
* **GitHub CLI:**
  * Installation and Setup
  * Repository / Issue Management
  * Pull Requests
  * Use in Automation
* **GitHub Actions:**
  * YAML Syntax & Workflow Context
  * Workflow Triggers & Scheduled Workflows
  * Workflow Runners & Workflow Status
  * Secrets and Env Vars
  * Caching Dependencies & Storing Artifacts
  * Marketplace Actions

## 13. Các chủ đề Git nâng cao (Advanced Git Topics)
* Git Reflog
* Git Bisect
* Git Worktree
* Git Attributes
* Git LFS (Large File Storage)

## 14. Công cụ cho Nhà phát triển GitHub (GitHub Developer Tools)
* **GitHub API:** REST API & GraphQL API
* **Creating Apps:** GitHub Apps & OAuth Apps
* **Webhooks**

## 15. Các tính năng bổ sung của GitHub (More GitHub Features)
* **GitHub Pages:** Deploying Static Websites, Custom Domains, Static Site Generators
* GitHub Actions / Packages / Gists
* GitHub Codespaces
* GitHub Marketplace / Models / Copilot
* GitHub Sponsors
* GitHub Security
* **GitHub Education:** Student Developer Pack, GitHub Classroom, Campus Program