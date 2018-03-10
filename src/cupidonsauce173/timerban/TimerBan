<?php

namespace cupidonsauce173\timerban;

use pocketmine\command\Command;
use pocketmine\command\CommandSender;
use pocketmine\permission\BanEntry;
use pocketmine\plugin\PluginBase;
use pocketmine\event\Listener;
use pocketmine\event\player\PlayerPreLoginEvent;
use pocketmine\utils\Config;
use pocketmine\Player;
use pocketmine\utils\TextFormat;

class TimerBan extends PluginBase implements Listener{
	private $banList;
	private $ipBanList;
	
	public function onEnable(){
		if(!file_exists($this->getDataFolder())){
			mkdir($this->getDataFolder());
		}
		
		$this->banList = new Config($this->getDataFolder()."BanList.yml", Config::YAML);
		$this->ipBanList = new Config($this->getDataFolder()."IPBanList.yml", Config::YAML);
		$this->getServer()->getPluginManager()->registerEvents($this, $this);
	}

	/**
	 * @param PlayerPreLoginEvent $event
     */
	public function onPlayerLogin(PlayerPreLoginEvent $event){
		$player = $event->getPlayer();
		$now = time();
		if($this->banList->exists(strtolower($player->getName()))){
			if($this->banList->get($player->getName()) > $now){
				$player->close("", "You are banned.");
				$event->setCancelled();
			}else{
				$this->banList->remove($player->getName());
			}
			return;
		}
		if($this->ipBanList->exists($player->getAddress())){
			if($this->ipBanList->get($player->getAddress()) > $now){
				$player->close("", "You are perm banned.");
				$event->setCancelled();
			}else{
				$this->ipBanList->remove($player->getAddress());
			}
		}
	}

	/**
	 * @param CommandSender $sender
	 * @param Command $command
	 * @param string $label
	 * @param array $params
	 * @return bool
     */
	public function onCommand(CommandSender $sender, Command $command, $label, array $params) : bool{
		switch($command->getName()){
			case "timerban":
				$sub = array_shift($params);
				switch($sub){
					case "add":
					$player = array_shift($params);
					$after = array_shift($params);
					$reason = implode(" ", $params);
					if(trim($player) === "" or !is_numeric($after)){
						$sender->sendMessage("•§5[Server]§f Usage: /timerban add §b<player> §e<time> [reason..]");
						break;
					}
					$after = round($after, 2);
					$secAfter = $after*3600;
					
					$due = $secAfter + time();
					
					$this->banList->set(strtolower($player), $due);
					$this->banList->save();
					
					$this->getServer()->broadcastMessage(TextFormat::WHITE . "•" . TextFormat::LIGHT_PURPLE . "[Server] " .TextFormat::GREEN . $player . " has been banned ". TextFormat::RED . $after . " hour(s) " . TextFormat::WHITE . "for " . TextFormat::RED . $reason . TextFormat::WHITE . ".");
					
					if(($player = $this->getServer()->getPlayer($player)) instanceof Player){
						$player->kick("•You have been banned for $after hour(s).");
					}
					break;
					case "remove":
					case "pardon":
					$player = array_shift($params);
					
					if(trim($player) === ""){
						$sender->sendMessage("•§5[Server] §fUsage: /timerban remove §e<player>");
						break;
					}
					
					if(!$this->banList->exists($player)){
						$sender->sendMessage("•§5[Server] §fThere is §4no player named §r\"$player\"");
						break;
					}
					
					$this->banList->remove($player);
					$this->banList->save();
					$sender->sendMessage("•§5[Server] §e\"$player\" §fhave been removed from the ban list.");
					$this->getServer()->broadcastMessage(TextFormat::WHITE . "•" . TextFormat::LIGHT_PURPLE . "[Server] " . TextFormat::GREEN . $player . " has been unbanned." );
					break;
					case "list":
					$list = $this->banList->getAll();
					$output = "•§aBan list §f: §b\n";
					foreach($list as $key => $due){
						$output .= $key.", ";
					}
					$output = substr($output, 0, -2);
					$sender->sendMessage($output);
					break;
					default:
					$sender->sendMessage("•§5[Server] §fUsage: ".$command->getUsage());
				}
				break;
			case "timerbanip":
				$sub = array_shift($params);
				switch($sub){
					case "add":
					$ip = array_shift($params);
					$after = array_shift($params);
					$reason = implode(" ", $params);
					if(trim($ip) === "" or !is_numeric($after)){
						$sender->sendMessage("•§5[Server] §fUsage: /timerban §e<player> §b<time> §6[reason..]");
						break;
					}
					$after = round($after, 2);
					$secAfter = $after*3600;
					if(filter_var($ip, FILTER_VALIDATE_IP)){
						foreach($this->getServer()->getOnlinePlayers() as $player){
							if($player->getAddress() === $ip){
								$player->kick("You have been banned for $after hour(s)for $reason.");
								break;
							}
						}
					}else{
						$player = $this->getServer()->getPlayer($ip);
						if($player instanceof Player){
							$ip = $player->getAddress();
							$player->kick("You have been banned for $after hour(s) for $reason.");
						}
					}

					$due = $secAfter + time();
					
					$this->ipBanList->set($ip, $due);
					$this->ipBanList->save();
					
					$sender->sendMessage("•§5[Server] §e$ip §fhas been banned for $after hours.");
					break;
					case "remove":
					case "pardon":
					$player = array_shift($params);
					
					if(trim($player) === ""){
						$sender->sendMessage("[•§5[Server] §fUsage: /timerban remove <player>");
						break;
					}
					
					if(!$this->ipBanList->exists($player)){
						$sender->sendMessage("•§5[Server] §fThere is no §eplayer §fwith IP§2 \"$player\"");
						break;
					}
					
					$this->ipBanList->remove($player);
					$this->ipBanList->save();
					$sender->sendMessage("•§5[Server] §2\"$player\" §fhave been §eremoved §ffrom the ban list.");
					break;
					case "list":
					$list = $this->ipBanList->getAll();
					$output = "IP Ban list : \n";
					foreach($list as $key => $due){
						$output .= $key.", ";
					}
					$output = substr($output, 0, -2);
					$sender->sendMessage($output);
					break;
					default:
					$sender->sendMessage("•§5[Server] Usage: ".$command->getUsage());
				}
		}
		return true;
	}
}
