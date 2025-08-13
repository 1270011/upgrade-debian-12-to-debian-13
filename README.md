# 🚀 Debian 12 (Bookworm) to Debian 13 (Trixie) Upgrade Guide

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Debian Version](https://img.shields.io/badge/Debian-12%20→%2013-red.svg)](https://www.debian.org/)
> A comprehensive, battle-tested guide for upgrading Debian 12 (Bookworm) to Debian 13 (Trixie) with minimal downtime and maximum safety.

## 🌟 Why This Guide?

This guide has been crafted through real-world experience and community feedback. It includes not just the upgrade steps, but also troubleshooting tips, automation options, and edge cases you might encounter. **Star this repo** if it helps you! ⭐

## ⚠️ Before You Begin

**🔴 CRITICAL: Always create a backup before proceeding!** Even though Debian upgrades are generally reliable, hardware failures, power outages, or unexpected issues can occur. Don't be the person who skips backups and regrets it later.

## 📋 Prerequisites Checklist

- [ ] System running Debian 12 (Bookworm)
- [ ] Root or sudo access
- [ ] Stable internet connection
- [ ] At least 10GB free space in root partition (recommended)
- [ ] Time for the upgrade process (30-90 minutes depending on system)

## 🛡️ Step 1: Create a System Backup

Choose one of these backup methods:

### Option A: Using `dd` (Full disk clone)
```bash
# Replace /dev/sdX with your actual disk device
sudo dd if=/dev/sdX of=/path/to/backup/disk-backup-$(date +%Y%m%d).img bs=4M status=progress
```

### Option B: Using Clonezilla (Recommended for beginners)
1. Download Clonezilla Live ISO
2. Boot from USB/CD
3. Follow the GUI to create a full system image

### Option C: Using rsync (File-level backup)
```bash
# Backup to external drive
sudo rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /path/to/backup/
```

## 💾 Step 2: Check Available Disk Space

Ensure you have sufficient space for the upgrade:

```bash
# Check root partition space
df -h /

# As a rule of thumb, ensure at least 10GB free space
# You can get by with less, but 10GB provides a comfortable buffer
```

If space is insufficient:
```bash
# Clean package cache
sudo apt clean

# Remove old kernels (keep current + 1 previous)
sudo apt autoremove --purge
```

## 🔄 Step 3: Update Current System

Before upgrading to Debian 13, ensure your Debian 12 system is fully up-to-date:

```bash
# Update package lists and upgrade all packages
sudo apt update && sudo apt full-upgrade

# Clean package cache to free up space
sudo apt clean

# Remove unnecessary packages
sudo apt autoremove
```

Reboot if kernel updates were installed:
```bash
sudo reboot
```

## 📝 Step 4: Backup Sources Configuration

Always backup your current sources configuration:

```bash
# Backup the main sources file
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

# Also backup sources.list.d directory if it exists
if [ -d "/etc/apt/sources.list.d" ]; then
    sudo cp -r /etc/apt/sources.list.d /etc/apt/sources.list.d.bak
fi
```

## 🔧 Step 5: Update Repository Sources

### Check Your Current Sources Configuration

First, determine which configuration method your system uses:

```bash
# Check if using traditional sources.list
if [ -s "/etc/apt/sources.list" ] && grep -q "bookworm" /etc/apt/sources.list; then
    echo "Using traditional sources.list format"
    SOURCES_METHOD="traditional"
elif find /etc/apt/sources.list.d -name "*.list" 2>/dev/null | xargs grep -l "bookworm" | head -1; then
    echo "Using sources.list.d directory format"
    SOURCES_METHOD="directory"
else
    echo "Please check your sources configuration manually"
fi
```

### Update Sources Based on Your Configuration

#### Method A: Traditional sources.list
```bash
# Option 1: Using sed (automated)
sudo sed -i 's/bookworm/trixie/g' /etc/apt/sources.list

# Option 2: Using apt edit-sources (interactive)
sudo apt edit-sources

# Option 3: Manual editing with vim/nano
sudo vim /etc/apt/sources.list
# Replace all instances of 'bookworm' with 'trixie'
```

#### Method B: sources.list.d directory
```bash
# Update all .list files in sources.list.d
find /etc/apt/sources.list.d -name "*.list" -exec sudo sed -i 's/bookworm/trixie/g' {} \;
```

### Verify Sources Update
```bash
# Check that bookworm has been replaced with trixie
grep -r "bookworm\|trixie" /etc/apt/sources.list /etc/apt/sources.list.d/
```

## 🔄 Step 6: Perform the Upgrade

### Update Package Lists
```bash
sudo apt update
```

### Review Upgradeable Packages (Recommended for Production)
```bash
# See what packages will be upgraded
apt list --upgradeable

# Count total packages to be upgraded
apt list --upgradeable | wc -l
```

### Minimal Upgrade First
```bash
# Upgrade without installing new packages
sudo apt upgrade --without-new-pkgs
```

### Simulate Full Upgrade
```bash
# Always simulate first to catch potential issues
sudo apt full-upgrade -s
```

**Review the simulation output carefully!** Look for:
- Packages being removed unexpectedly
- Conflicts or dependency issues
- Services that might need restart

### Perform Full Upgrade
If the simulation looks good, proceed:

```bash
sudo apt full-upgrade
```

**Note:** This process may take 30-90 minutes depending on your system and internet connection. Also you usually want to "baby-sit" the process to identify possible errors and read carefully the news / changelogs to stay up-to-date with the new changes in debian 13. Do not skip reading the news for critical systems.

## ✅ Step 7: Verify Upgrade Success

Check your new Debian version:
```bash
# Should output "13" or "trixie/sid"
cat /etc/debian_version

# Alternative verification
lsb_release -a
```

## 🆕 Step 8: Modernize to DEB822 Format (Optional)

Debian 13 introduces the new DEB822 sources format. Modernize your sources:

```bash
# Convert to modern sources format
sudo apt modernize-sources
```

This command will:
- Convert traditional sources.list to the new format
- Place new sources in `/etc/apt/sources.list.d/`
- Backup old configuration automatically

## 🤖 Automation with Ansible

For managing multiple machines, use Ansible playbook.


## 🐛 Troubleshooting & Known Issues

### Issue 1: GTK/Qt Theme Inconsistencies

**Symptoms:** After upgrade, Qt applications may display with different themes than GTK applications, especially noticeable in minimal installations and custom, not default themes.

**Solution:**
```bash
# Install Qt5 Kvantum theme engine
sudo apt install qt5-style-kvantum

# Create Kvantum config directory
mkdir -p ~/.config/Kvantum

# Copy your current theme to Kvantum
# Replace 'myFooBarTheme' with your actual theme name
cp -r ~/.themes/myFooBarTheme/Kvantum/* ~/.config/Kvantum/
# Or from system themes:
# cp -r /usr/share/themes/myFooBarTheme/Kvantum/* ~/.config/Kvantum/

# Configure Kvantum theme
kvantummanager

# Set environment variables for consistent theming
echo "QT_QPA_PLATFORMTHEME=qt5ct" >> ~/.profile
echo "QT_STYLE_OVERRIDE=kvantum" >> ~/.profile

# Or set system-wide in /etc/environment:
echo -e "QT_QPA_PLATFORMTHEME=qt5ct\nQT_STYLE_OVERRIDE=kvantum" | sudo tee -a /etc/environment
```

**Note:** This is primarily needed for minimal desktop installations and shouldn't affect headless servers.

### Issue 2: XFCE Workspace Navigation Shortcuts

**Symptoms:** Ctrl+Alt+Up/Down shortcuts for workspace navigation stop working in XFCE.

**Quick Fix Script:**
```bash
#!/bin/bash
# Save as ~/fix-xfce-workspaces.sh

echo "Applying XFCE workspace navigation hotfix..."

# Find the correct plugin number for workspace pager
PLUGIN_NUM=$(xfconf-query -c xfce4-panel -l | while read prop; do
    if echo "$prop" | grep -q plugin; then
        value=$(xfconf-query -c xfce4-panel -p "$prop" 2>/dev/null)
        if echo "$value" | grep -qi "pager\|workspace"; then
            echo "$prop" | sed 's/.*plugin-\([0-9]*\).*/\1/'
        fi
    fi
done | head -1)

if [ -n "$PLUGIN_NUM" ]; then
    echo "Found workspace pager at plugin-$PLUGIN_NUM"
    
    # Toggle miniature view to reset the plugin
    xfconf-query -c xfce4-panel -p "/plugins/plugin-$PLUGIN_NUM/miniature-view" -s true
    sleep 1
    xfconf-query -c xfce4-panel -p "/plugins/plugin-$PLUGIN_NUM/miniature-view" -s false
    
    echo "Done! Try your Ctrl+Alt+Up/Down shortcuts now."
else
    echo "Could not find workspace pager plugin. Please check manually."
fi
```

**Make it executable and run:**
```bash
chmod +x ~/fix-xfce-workspaces.sh
~/fix-xfce-workspaces.sh
```

**Auto-run on login:**
Add to your `~/.profile`:
```bash
# Add this line to ~/.profile
~/fix-xfce-workspaces.sh 2>/dev/null
```

## 🆘 Recovery Options

If something goes wrong during upgrade:

### Boot Issues
1. Boot from Debian Live USB
2. Mount your system partition
3. Restore from backup or chroot to fix issues

### Package Conflicts
```bash
# Fix broken packages
sudo apt --fix-broken install

# Reconfigure packages
sudo dpkg --configure -a

# Force package installation if needed (use carefully!)
sudo apt install -f
```

### Rollback Sources
```bash
# Restore original sources
sudo cp /etc/apt/sources.list.bak /etc/apt/sources.list
sudo apt update
```

## 📊 Performance Impact

Expected upgrade statistics:
- **Packages upgraded:** ~900-3000 (depending on installation)
- **Download size:** 800MB - 2GB
- **Disk space required:** 2-5GB during upgrade
- **Time required:** 30-90 minutes
- **Reboot required:** Usually yes (kernel updates)

## 🤝 Contributing

Found an issue or have improvements? Please:
1. ⭐ Star this repository
2. 🐛 Open an issue
3. 🔧 Submit a pull request
4. 💬 Share your experience in discussions

## 📚 Additional Resources

- [Official Debian Release Notes](https://www.debian.org/releases/trixie/releasenotes)
- [Debian Administration Handbook](https://debian-handbook.info/)
- [Debian Package Management](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html)

## ⚖️ License

This guide is released under the MIT License. Feel free to use, modify, and distribute.

---

**⭐ If this guide helped you successfully upgrade to Debian 13, please star this repository!** Your stars help others discover this resource and motivate continued maintenance and improvements.

**🐛 Encountered issues?** Please report them in the Issues section so we can help others avoid the same problems.

**💡 Have improvements?** Pull requests are welcome! Let's make this the best Debian upgrade guide available.

---
*Last updated: August 2025 | Tested on: Debian 12.x → 13.x*
