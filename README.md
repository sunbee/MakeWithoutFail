# Make Without Fail
Build resilience in Raspberry Pi-based compute infrastructure for makers. Start with setting up Pi to operate off an SSD instead of the conventional SD card. The build resilience by containerizing apps with Docker, automated backups of application data to Cloud and disk imaging.

This note covers automated backups of application data to Cloud with **rclone** and **rsync**.

## Checklist

Step | Instruction
---------- | -----------
YAML file for IOT stack | See Andreas Speiss and Graham Garner here. The application data is mapped to a volumes folder on host machine.
