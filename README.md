BukkitUpdateChecker
===================

Provides a service to handle checking if the current plugin calling is already the latest on the bukkit dev site.

This service requires that:

    1. plugin filename on disk (when downloaded) uses a specific (yet flexible) format
    2. bukkit dev package name uses a specific format

The specific format is:

    (on disk)   <Name>-<Major>.<Minor>.<Revision><Tag>
    (dev site)  <Major>.<Minor>.<Revision><Tag>

For example:

    (on disk)   KFactions-1.34.67B     OR    KFactions-4.54.123456789LETTERSATEND
    (dev site)  1.32.67B

== An Example Implementation ==

    /* this allows the update checker to create a thread to get the latest version */
    public void onEnable() {
      updateChecker = new UpdateChecker();
      updateChecker.setProjectName("kfactions");
      updateChecker.start();
    }

    /* this alerts anyone who has been OPPED if the plugin is out of date */
    @EventHandler
    public void onPlayerJoin(PlayerJoinEvent event) {
        Player          p;
        Version         thisVersion;
        Version         latestVersion;
        FactionPlayer   fp;
        Faction         f;
        
        p = event.getPlayer();
        
        if (p == null) {
            return;
        }
        
        if (!p.isOp()) {
            return;
        }
        
        if (updateChecker.canUpdate()) {
            thisVersion = updateChecker.getThisVersion();
            latestVersion = updateChecker.getLatestVersion();
            
            if (thisVersion.getMaj() == 0 &&
                thisVersion.getMin() == 0 &&
                thisVersion.getRev() == 0) {
                p.sendMessage("§d[KFactions] Can not get version from JAR filename. Did you rename the Jar? I am unable to alert you to updates.");
            } else {
                p.sendMessage(String.format(
                        "§d[%s] The newest version is §c%s§d and your version is §c%s§d. You are missing §c%d§d new features and §c%d§d update/fix patches.",
                        getName(),
                        latestVersion, thisVersion, latestVersion.getMin() - thisVersion.getMin(), latestVersion.getRev() - thisVersion.getRev()
                ));
            }
        }
    }
