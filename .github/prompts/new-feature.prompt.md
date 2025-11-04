---
mode: agent
description: Create a new feature 
---

# Create New Feature

## Step 0: Moe any existing story to complete
- if there are any folders within stories/current, inform the user to move them to stories/archive before creating a new feature.
## Step 1: Ask for Feature Name
If not provided, ask the user for the story number and feature name

## Step 2: Create File Structure
- Create folder in `stories/current/[story-number]-[feature-name]`
- Create story.md, technical.md files using templates in the `templates` directory and ask the user to provide content for each.
- Get confirmation from the user before proceeding.
- Once user confirms, create the spec.md with the specs using the story.md and technical.md content.
- Ask user the review the specs and confirm before proceeding.
- Upon confirmation, create the implementation.md file.
- Get confirmation from the user before proceeding to step 3

## Step 3: Implement the changes
- Use the implementation.md file to guide the implementation of the feature.