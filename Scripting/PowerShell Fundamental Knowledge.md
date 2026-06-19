## ## **From My [[AI Server/4) Deploying Agent on A Windows Server|Windows Vector Agent Conf]]** :

- @' '@ is used like a "cat << EOF > MyFile.txt" it's a Here-string, where cat EOF is a Here-Doc
- | Out-File -Encoding UTF8 "C:\vector\config\agent.toml" is used to set the encoding and the destination file. The "NoBom" we can encounter in utf8NoBOM means Byte Order Mark but sometimes, some agents doesn't accept it.
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
- [transforms.enrich_windows]
	type = "remap"
	inputs = ["win_security", "win_system", "win_application", "win_updates"]
	- Take in inputs previous defined variables
- source = '''
	.hostname = get_hostname!()   # Récupère dynamiquement le nom de la machine.
	.source_os = "windows"        # Ajoute un tag fixe pour identifier l'OS.
	.site = "saint-chamond"       # Ajoute un tag fixe pour identifier le site géographique.
	
	# Détermination du type de log basé sur le nom du canal
	.log_type = if exists(.channel) {
	  ch = downcase(to_string!(.channel))
	  if contains(ch, "security") { "security" }
	  else if contains(ch, "system") { "system" }
	  else if contains(ch, "application") { "application" }
	  else if contains(ch, "windowsupdate") { "windows_update" }
	  else { "windows_generic" }
	} else { "windows_generic" }
	
	# Normalisation de la sévérité (level) du log
	level_raw = downcase(to_string(.level) ?? "info")
	.level = if includes(["critical", "error"], level_raw) { "error" }
	  else if includes(["warning", "warn"], level_raw) { "warning" }
	  else { "info" }
	
	# Extraction et conversion de l'Event ID (EID)
	eid = to_int(.system.event_id.value) ?? 0
	.event_id = to_string(eid)
	
	# Sur-classement de la sévérité en "critical" si l'Event ID est sensible (ex: échec de connexion, log effacé)
	.level = if includes([4625, 4740, 1102, 4648, 4719, 4964, 1074, 6008], eid) { "critical" }
	  else { .level }
	
	# Nettoyage du message de l'événement
	.message = to_string(.message) ?? to_string(.rendered_message) ?? "no message"
	
	# Ajout d'un horodatage si absent
	if !exists(.timestamp) { .timestamp = now() }
	- Everything has been told in my natural langage. But to resume : First a Metadata collect, Second a Log-type classification with a basic and a critical filters, Third a level hierarchy (no need to define because Vector, before each entry, has already converts JSON and we have only filtered one type, but level is another one we just called), Forth an Extraction and EID Treatment, Fifth a Critical-security-filter in case level triggers, Sixth it keeps the explicative message and put a timestamp or keeps the original, Seventh it takes data from inventory_http (we will define later), Eighth a anti-noise filter and Nineth the Both sinks to drop right data in 
