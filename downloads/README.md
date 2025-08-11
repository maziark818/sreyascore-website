# SreyasBridge Downloads

This directory contains the official SreyasBridge download packages.

## Available Packages

### 1. SreyasBridge-QuickStart.zip
**Recommended for new users**
- Main bridge server script
- Web interface script  
- Startup batch files
- Quick start guide
- Basic configuration

**Size:** ~50 KB
**Contents:**
```
SreyasBridge-QuickStart/
├── SreyasBridge.ps1          # Main HTTP bridge server
├── SreyasBridge-Web.ps1      # Web interface server
├── Start-SreyasBridge.bat    # Easy startup script
├── Start-SreyasBridge-Web.bat # Web interface startup
├── README.md                  # Quick start instructions
└── config.json               # Basic configuration
```

### 2. SreyasBridge-Complete.zip
**Full package with all features**
- All PowerShell scripts
- Enhanced web interface
- Command builder and templates
- System diagnostics tools
- Complete documentation
- Example custom commands
- Advanced configuration options

**Size:** ~200 KB
**Contents:**
```
SreyasBridge-Complete/
├── Core Scripts/
│   ├── SreyasBridge.ps1
│   ├── SreyasBridge-Web.ps1
│   ├── Manage-SreyasBridge.ps1
│   └── Fix-F.ps1
├── Web Interface/
│   ├── index.html
│   ├── styles.css
│   └── script.js
├── Startup Scripts/
│   ├── Start-SreyasBridge.bat
│   ├── Start-SreyasBridge-Web.bat
│   └── Start-SreyasBridge-Simple.ps1
├── Documentation/
│   ├── README.md
│   ├── QUICK-START-GUIDE.md
│   └── API-REFERENCE.md
├── Examples/
│   ├── Custom-Commands.ps1
│   └── Templates/
├── Configuration/
│   ├── config.json
│   └── security.json
└── Tools/
    ├── System-Diagnostics.ps1
    └── Performance-Monitor.ps1
```

### 3. Source Code
**For developers and contributors**
- Full source code repository
- Development documentation
- Build scripts
- Testing framework
- Contribution guidelines

**Access:** https://github.com/sreyascore/sreyasbridge

## Installation Instructions

### Quick Start Package
1. Download `SreyasBridge-QuickStart.zip`
2. Extract to `C:\Scripts\` (or your preferred location)
3. Right-click `Start-SreyasBridge.bat` → "Run as Administrator"
4. Right-click `Start-SreyasBridge-Web.bat` → "Run as Administrator"
5. Open browser to `http://localhost:8080`

### Complete Package
1. Download `SreyasBridge-Complete.zip`
2. Extract to `C:\Scripts\`
3. Run `Manage-SreyasBridge.ps1` for guided setup
4. Follow the interactive configuration wizard
5. Access web interface at `http://localhost:8080`

## System Requirements

- **OS:** Windows 10/11 or Windows Server 2016+
- **PowerShell:** 5.1 or later
- **RAM:** 2 GB minimum (4 GB recommended)
- **Storage:** 100 MB free space
- **Network:** Local network access for remote management

## Security Notes

- SreyasBridge runs on localhost by default
- For remote access, configure firewall rules appropriately
- Use HTTPS in production environments
- Regularly update to latest versions
- Review security configuration before deployment

## Support

- **Documentation:** https://sreyascore.net/docs
- **GitHub Issues:** https://github.com/sreyascore/sreyasbridge/issues
- **Community:** https://discord.gg/sreyascore
- **Email:** support@sreyascore.net

## Version History

### v2.1.0 (Latest)
- Enhanced web interface
- Improved error handling
- New custom command builder
- Performance optimizations
- Security improvements

### v2.0.0
- Complete rewrite with modern architecture
- RESTful API design
- Responsive web interface
- Enhanced security features

### v1.0.0
- Initial release
- Basic HTTP bridge functionality
- Simple web interface
- Core system management features

## License

SreyasBridge is released under the MIT License. See LICENSE file for details.

---

**Need help?** Check our [Quick Start Guide](../docs/quick-start.html) or [contact support](mailto:support@sreyascore.net). 