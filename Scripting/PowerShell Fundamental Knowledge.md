## **From My [AGe](%20Deploying%20Agent%20on%20a%20Windows%20Server.md):

- @' '@ is used like a "cat << EOF > MyFile.txt" it's a Here-string, where cat EOF is a Here-Doc
- | Out-File -Encoding utf8NoBOM "C:\vector\config\agent.toml" is used to set the encoding and the destination file. The "NoBom" is designed to be a Byte Order Mark but sometimes, some agents doesn't accept it.