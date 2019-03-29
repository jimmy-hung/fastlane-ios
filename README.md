add this to your fastlane/Fastfile

```
import_from_git(
  url: "https://github.com/jimmy-hung/fastlane-ios", # The URL of the repository to import the Fastfile from.
  path: "Fastfile" # The path of the Fastfile in the repository. Defaults to fastlane/Fastfile.
)
```
