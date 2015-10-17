require "yaml"
require "date"

conf = YAML.load_file(".grav-pull.yml")

task default: %w[pull]

task :pull do
  puts "Creating remote backup..."
  move_to_address = ' "cd ' + conf["remote"]["path"]
  create_backup = ' && bin/grav backup"'
  ssh = "ssh " + conf["ssh"]["username"] + "@" + conf["ssh"]["host"] +
    " -p " + conf["ssh"]["port"] + move_to_address + create_backup
  sh(ssh)

  # Make sure you remove "backup" in web.config + .htaccess:
  # RewriteRule ^(.git|cache|bin|logs|backup)/(.*) error [L]?
  # Add to .htaccess?
  # allow zip from backup?

  # Remove existing backups
  puts "Remove existing backups..."
  sh("rm -rf backup")

  # Download backup
  puts "Downloading backup..."
  scp = "scp -P 4000 " + conf["ssh"]["username"] + "@" + conf["ssh"]["host"] +
    ":" + conf["remote"]["path"] + "/backup/* ."
  sh(scp)

  # Remove backup
  puts "Removing backup..."
  move_to_backup = ' "cd ' + conf["remote"]["path"] + '/backup'
  remove_backup = ' && rm -r * -f"'
  ssh = "ssh " + conf["ssh"]["username"] + "@" + conf["ssh"]["host"] +
    " -p " + conf["ssh"]["port"] + move_to_backup + remove_backup
  sh(ssh)

  # Extracting backup
  puts "Extracting backup..."
  sh("mkdir backup")
  sh("find . -depth -name '*.zip' -exec unzip -d backup '{}' ';'")
  sh("find . -depth -name '*.zip' -exec rm -rf '{}' ';'")

  # Copy over user/
  puts "Copying remote data..."
  sh("find . -depth -name 'backup' -exec cd '{}' ';'  && cp -rf user ../")

  # Remove the backup
  puts "Removing backup..."
  sh("find . -depth -name 'backup' -exec rm -rf '{}' ';'")

  # Migration completed
  puts "Migration completed!"
end
