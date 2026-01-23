#!/bin/bash
#
# notmcp installer
# Run with: curl -fsSL notmcp.com/use | bash
#
set -e

SKILL_DIR="$HOME/.claude/skills/notmcp"
REPO="DomVinyard/NotMCP"
BRANCH="main"
REPO_URL="https://github.com/$REPO/archive/refs/heads/$BRANCH.tar.gz"

echo "Installing notmcp..."

# Create skill directory structure
mkdir -p "$SKILL_DIR/bin"
mkdir -p "$SKILL_DIR/scripts"

# Download and extract the skill folder
echo "Downloading from $REPO..."
curl -fsSL "$REPO_URL" | tar -xz --strip-components=2 -C "$SKILL_DIR" "NotMCP-$BRANCH/skill"

# Make CLI executable
chmod +x "$SKILL_DIR/bin/notmcp"

# Initialize empty credentials file with secure permissions
if [ ! -f "$SKILL_DIR/.credentials" ]; then
    echo "{}" > "$SKILL_DIR/.credentials"
fi
chmod 600 "$SKILL_DIR/.credentials"

echo ""
echo "notmcp installed successfully!"
echo ""
echo "Location: $SKILL_DIR"
echo ""
echo "Your agent can now use notmcp. Try asking it to:"
echo "  - List available tools"
echo "  - Create a tool for an API you use"
echo "  - Connect to a service like PostHog, Stripe, or Gmail"
echo ""
