# HomePlugin
package me.example.homeplugin;

import org.bukkit.Bukkit;
import org.bukkit.Location;
import org.bukkit.World;
import org.bukkit.command.*;
import org.bukkit.configuration.file.FileConfiguration;
import org.bukkit.entity.Player;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.HashMap;
import java.util.UUID;

public class HomePlugin extends JavaPlugin {

    private FileConfiguration config;
    private final HashMap<UUID, Long> cooldowns = new HashMap<>();
    private final long COOLDOWN_TIME = 5000; // 5 seconds

    @Override
    public void onEnable() {
        saveDefaultConfig();
        config = getConfig();
    }

    private boolean inCooldown(Player player) {
        UUID uuid = player.getUniqueId();
        if (!cooldowns.containsKey(uuid)) return false;

        return System.currentTimeMillis() - cooldowns.get(uuid) < COOLDOWN_TIME;
    }

    private void setCooldown(Player player) {
        cooldowns.put(player.getUniqueId(), System.currentTimeMillis());
    }

    @Override
    public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args) {

        if (!(sender instanceof Player player)) return true;

        String uuid = player.getUniqueId().toString();

        // /sethome
        if (cmd.getName().equalsIgnoreCase("sethome")) {

            config.set(uuid + ".world", player.getLocation().getWorld().getName());
            config.set(uuid + ".x", player.getLocation().getX());
            config.set(uuid + ".y", player.getLocation().getY());
            config.set(uuid + ".z", player.getLocation().getZ());
            config.set(uuid + ".yaw", player.getLocation().getYaw());
            config.set(uuid + ".pitch", player.getLocation().getPitch());

            saveConfig();
            player.sendMessage("§aHome set!");
            return true;
        }

        // /home
        if (cmd.getName().equalsIgnoreCase("home")) {

            if (inCooldown(player)) {
                player.sendMessage("§cCooldown active!");
                return true;
            }

            if (!config.contains(uuid + ".world")) {
                player.sendMessage("§cYou don't have a home set!");
                return true;
            }

            World world = Bukkit.getWorld(config.getString(uuid + ".world"));
            double x = config.getDouble(uuid + ".x");
            double y = config.getDouble(uuid + ".y");
            double z = config.getDouble(uuid + ".z");
            float yaw = (float) config.getDouble(uuid + ".yaw");
            float pitch = (float) config.getDouble(uuid + ".pitch");

            Location home = new Location(world, x, y, z, yaw, pitch);
            player.teleport(home);

            setCooldown(player);
            player.sendMessage("§aTeleported home!");
            return true;
        }

        // /delhome
        if (cmd.getName().equalsIgnoreCase("delhome")) {
            config.set(uuid, null);
            saveConfig();
            player.sendMessage("§cHome deleted!");
            return true;
        }

        return false;
    }
}
