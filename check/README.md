# Namada Check Status via Telegram

https://t.me/mychecknambot

## Install Node.js
```
sudo apt update
sudo apt install nodejs npm
```
## Install Dependencies
```
npm install node-telegram-bot-api
```
## Create Folder & File
```
mkdir status
cd status
```
```
nano status.js
```
```
const TelegramBot = require('node-telegram-bot-api');
const { exec } = require('child_process');

const token = 'heretokentelegram';

const bot = new TelegramBot(token, { polling: true });

bot.on('message', async (msg) => {
    const chatId = msg.chat.id;
    const messageText = msg.text;

    switch (messageText) {
        case '/start':
            bot.sendMessage(chatId, 'Project:[adamittem] Please choose', {
                reply_markup: {
                    keyboard: [['Check MyNODE'], ['Check MyVPS']],
                    resize_keyboard: true,
                },
            });
            break;
        case 'Check MyNODE':
            exec('curl -s http://127.0.0.1:26657/status | jq \'.result.sync_info.latest_block_height, .result.node_info.network, .result.node_info.version, .result.validator_info.address, .result.sync_info.catching_up\'', (error, stdout, stderr) => {
                if (error) {
                    bot.sendMessage(chatId, `Error executing command: ${error.message}`);
                    return;
                }
                if (stderr) {
                    bot.sendMessage(chatId, `Error in response: ${stderr}`);
                    return;
                }
                try {
                    const [latestBlockHeight, network, version, validatorAddress, catchingUp] = stdout.trim().split('\n').map(item => item.replace(/"/g, ''));
                    const status = catchingUp.trim() === 'false' ? 'Online' : 'Offline';
                    const formattedOutput = `
Status              : ${status}
Latest Block  : ${latestBlockHeight.trim()}
Chain               : ${network.trim()}
Version           : ${version.trim()}
Validator        : ${validatorAddress.trim()}`;
                    bot.sendMessage(chatId, formattedOutput);
                } catch (parseError) {
                    bot.sendMessage(chatId, 'Error parsing status data.');
                }
            });
            break;
        case 'Check MyVPS':
            exec('top -bn1 | grep "Cpu(s)" | awk \'{print $2 + $4}\' && free -m | awk \'FNR == 2 {print $3/$2 * 100}\' && df -h | awk \'$NF=="/" {print $5}\'', (error, stdout, stderr) => {
                if (error) {
                    bot.sendMessage(chatId, `Error executing command: ${error.message}`);
                    return;
                }
                if (stderr) {
                    bot.sendMessage(chatId, `Error in response: ${stderr}`);
                    return;
                }
                try {
                    const [cpuUsage, ramUsage, ssdUsage] = stdout.trim().split('\n');
                    const formattedOutput = `
Load CPU      : ${cpuUsage.trim()}%
Usage RAM  : ${ramUsage.trim()}%
Usage SSD   : ${ssdUsage.trim()}`;
                    bot.sendMessage(chatId, formattedOutput);
                } catch (parseError) {
                    bot.sendMessage(chatId, 'Error parsing system usage data.');
                }
            });
            break;
        default:
            bot.sendMessage(chatId, 'Invalid option, please choose again.');
            break;
    }
});

```

- Create Telegram Bot in Botfather
- Import Telegram bot token in the code

## RUN
```
node status.js
```
## Telegram
/start
```


