- Solution explorer : provides logical view , hides Build artifacts (bin, obj)
- Folder structure: Physical representation on disk, shows all hidden files .bin, .obj
- **ProjectName.csproj**: XML file that defines .NET version, specifies framework configurations
- **bin & obj:** bin contains compiled files (DDL), obj temporary files to speedup building artifacts
- Add bin and obj to .gitignore
- **launchsetting.json :** inside properties/ , defines urls, environment variables (this is only used for development)
## AppSettings vs Launch settings:
- App settings is used to configure settings that will be used when program is built (connection string, logging levels)
- App settings reduces the need to hardcoding values and settings
- LaunchSettings settings that launch app during development

