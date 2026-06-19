## ## **From My [[AI Server/4) Deploying Agent on A Windows Server|Windows Vector Agent Conf]]** :

- @' '@ is used like a "cat << EOF > MyFile.txt" it's a Here-string, where cat EOF is a Here-Doc
- | Out-File -Encoding utf8NoBOM "C:\vector\config\agent.toml" is used to set the encoding and the destination file. The "NoBom" is designed to be a Byte Order Mark but sometimes, some agents doesn't accept it.
- [sources.win_security]
  type = "windows_event_log"
  channels = ["Security"]

  [sources.win_system]
  type = "windows_event_log"
  channels = ["System"]

  [sources.win_application]
  type = "windows_event_log"
  channels = ["Application"]

  [sources.win_updates]
  type = "windows_event_log"
  channels = ["Microsoft-Windows-WindowsUpdateClient/Operational"]
     - It defines core blocs with the type = API listens and channels to separate each log flux
 - [sources.inventory_http]
	type = "http_server"
	address = "127.0.0.1:9001"
	decoding.codec = "json"
	- It opens a mini webserver only on localhost on port 9001 in case he received inventory data