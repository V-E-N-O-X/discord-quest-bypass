### Discord Quest Bypass Code!
<br>

```jslet wpRequire;
window.webpackChunkdiscord_app.push([[Math.random()], {}, (req) => { wpRequire = req; }]);

let ApplicationStreamingStore, RunningGameStore, QuestsStore, ExperimentStore, FluxDispatcher, api;

const buildId = window.GLOBAL_ENV.SENTRY_TAGS.buildId;
const useNewApi = buildId !== "366c746173a6ca0a801e9f4a4d7b6745e6de45d4";

const getModule = (condition, exportName = 'default') => 
    Object.values(wpRequire.c).find(condition)?.exports?.[exportName];

ApplicationStreamingStore = getModule(x => x?.exports?.default?.getStreamerActiveStreamMetadata, useNewApi ? 'Z' : 'default');
RunningGameStore = getModule(x => x?.exports?.default?.getRunningGames, useNewApi ? 'ZP' : 'default');
QuestsStore = getModule(x => x?.exports?.default?.getQuest, useNewApi ? 'Z' : 'default');
ExperimentStore = getModule(x => x?.exports?.default?.getGuildExperiments, useNewApi ? 'Z' : 'default');
FluxDispatcher = getModule(x => x?.exports?.default?.flushWaitQueue, useNewApi ? 'Z' : 'default');
api = getModule(x => x?.exports?.getAPIBaseURL, useNewApi ? 'tn' : 'HTTP');

const quest = [...QuestsStore.quests.values()].find(x => 
    x.id !== "1245082221874774016" && 
    x.userStatus?.enrolledAt && 
    !x.userStatus?.completedAt && 
    new Date(x.config.expiresAt).getTime() > Date.now()
);

const isApp = navigator.userAgent.includes("Electron/");
if (!isApp) {
    console.log("This no longer works in browser. Use the desktop app!");
} else if (!quest) {
    console.log("You don't have any uncompleted quests!");
} else {
    const pid = Math.floor(Math.random() * 30000) + 1000;
    let applicationId, applicationName, secondsNeeded, secondsDone, canPlay;

    const setupQuestDetails = () => {
        if (quest.config.configVersion === 1) {
            ({ applicationId, applicationName } = quest.config);
            secondsNeeded = quest.config.streamDurationRequirementMinutes * 60;
            secondsDone = quest.userStatus?.streamProgressSeconds ?? 0;
            canPlay = quest.config.variants.includes(2);
        } else if (quest.config.configVersion === 2) {
            ({ id: applicationId, name: applicationName } = quest.config.application);
            canPlay = ExperimentStore.getUserExperimentBucket("2024-04_quest_playtime_task") > 0 && quest.config.taskConfig.tasks["PLAY_ON_DESKTOP"];
            const taskName = canPlay ? "PLAY_ON_DESKTOP" : "STREAM_ON_DESKTOP";
            secondsNeeded = quest.config.taskConfig.tasks[taskName].target;
            secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0;
        }
    };

    const updateQuestProgress = (data, taskName) => {
        const progress = quest.config.configVersion === 1 
            ? data.userStatus.streamProgressSeconds 
            : Math.floor(data.userStatus.progress[taskName].value);

        console.log(`Quest progress: ${progress}/${secondsNeeded}`);

        if (progress >= secondsNeeded) {
            console.log("Quest completed!");
            if (canPlay) {
                removeFakeGame();
            } else {
                ApplicationStreamingStore.getStreamerActiveStreamMetadata = realFunc;
            }
            FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", updateQuestProgress);
        }
    };

    const spoofGame = async () => {
        const res = await api.get({ url: `/applications/public?application_ids=${applicationId}` });
        const appData = res.body[0];
        const exeName = appData.executables.find(x => x.os === "win32").name.replace(">", "");
        
        const fakeGame = {
            cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
            exeName,
            exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
            hidden: false,
            isLauncher: false,
            id: applicationId,
            name: appData.name,
            pid,
            pidPath: [pid],
            processName: appData.name,
            start: Date.now(),
        };
        
        RunningGameStore.getRunningGames().push(fakeGame);
        FluxDispatcher.dispatch({ type: "RUNNING_GAMES_CHANGE", removed: [], added: [fakeGame], games: RunningGameStore.getRunningGames() });
        
        FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", (data) => updateQuestProgress(data, "PLAY_ON_DESKTOP"));
        console.log(`Spoofed your game to ${applicationName}. Wait for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`);
    };

    const removeFakeGame = () => {
        const games = RunningGameStore.getRunningGames();
        const fakeGame = games.find(game => game.pid === pid);
        const idx = games.indexOf(fakeGame);
        if (idx > -1) {
            games.splice(idx, 1);
            FluxDispatcher.dispatch({ type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: [] });
        }
    };

    const spoofStream = () => {
        const realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata;
        ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
            id: applicationId,
            pid,
            sourceName: null
        });

        FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", (data) => updateQuestProgress(data, "STREAM_ON_DESKTOP"));
        console.log(`Spoofed your stream to ${applicationName}. Stream any window in vc for ${Math.ceil((secondsNeeded - secondsDone) / 60)} more minutes.`);
        console.log("Remember that you need at least 1 other person to be in the vc!");
    };

    setupQuestDetails();

    if (canPlay) {
        spoofGame();
    } else {
        spoofStream();
    }
}
```
