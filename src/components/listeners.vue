<script>
window.addEventListener('load', async function () {
  const marker = document.getElementById('indicator');
  const home = document.getElementById('home');
  const recent = document.getElementById('recent');
  const games = document.getElementById('games');
  const favs = document.getElementById('favs');
  const friends = document.getElementById('friends');
  const messages = document.getElementById('messages');
  const activity = document.getElementById('activity');
  const settings = document.getElementById('settings-popup');
  const settingsbackblur = document.getElementById('settings-backblur');
  const searchbars = document.querySelectorAll('div.search-bar > input[type="text"]');
  const fs = window.__TAURI__.fs;
  const path = window.__TAURI__.path;

  const data = JSON.parse(await fs.readTextFile(await path.appDir() + 'storage/cache/user/UserProfile.json', (err) => {
    if (err) throw err;
  }));
  document.getElementById('text').value = data.username;
  try {
    await fs.readBinaryFile(await path.appDir() + 'storage/cache/user/pfp.png');
    document.getElementById('output').src = window.__TAURI__.tauri.convertFileSrc(await path.appDir() + 'storage/cache/user/pfp.png') + `?${new Date().getSeconds()}`;
  } catch (err) {
    console.log(err);
  }

  await require('./launchers/find-games').getInstalledGames()
    .catch((err) => {
      return console.log(err);
    });

  await require('./launchers/find-games').loadGames('recentGamesListMainPage')
    .catch((err) => {
      return console.log(err);
    });

  document.getElementById('main-loading-overlay').style.opacity = '0';
  document.getElementById('main-loading-overlay').style.visibility = 'hidden';
  marker.style.top = '0px';

  document.getElementById('home-btn').addEventListener('click', async function () {
    home.style.display = 'flex';
    recent.style.display = 'none';
    games.style.display = 'none';
    favs.style.display = 'none';
    friends.style.display = 'none';
    messages.style.display = 'none';
    activity.style.display = 'none';

    marker.style.top = '0px';

    await require('./launchers/find-games').loadGames('recentGamesListMainPage')
      .catch((err) => {
        return console.log(err);
      });
  });

  document.getElementById('recent-btn').addEventListener('click', async function () {
    home.style.display = 'none';
    recent.style.display = 'flex';
    games.style.display = 'none';
    favs.style.display = 'none';
    friends.style.display = 'none';
    messages.style.display = 'none';
    activity.style.display = 'none';

    marker.style.top = '81px';
    marker.style.height = '30px';

    await require('./launchers/find-games').loadGames('recentGamesList')
      .catch((err) => {
        return console.log(err);
      });
  });

  document.getElementById('games-btn').addEventListener('click', async function () {
    marker.style.top = '131px';

    home.style.display = 'none';
    recent.style.display = 'none';
    games.style.display = 'flex';
    favs.style.display = 'none';
    friends.style.display = 'none';
    messages.style.display = 'none';
    activity.style.display = 'none';

    await require('./launchers/find-games').loadGames('allGamesList')
      .catch((err) => {
        return console.log(err);
      });

    document.getElementById('gamesInput').focus();
  });

  document.getElementById('favs-btn').addEventListener('click', async function () {
    marker.style.top = '180px';

    home.style.display = 'none';
    recent.style.display = 'none';
    games.style.display = 'none';
    favs.style.display = 'flex';
    messages.style.display = 'none';
    activity.style.display = 'none';
    friends.style.display = 'none';

    await require('./launchers/find-games').loadGames('favGamesList')
      .catch((err) => {
        return console.log(err);
      });

    document.getElementById('favsInput').focus();
  });

  document.getElementById('messages-btn').addEventListener('click', async function () {
    marker.style.top = '261px';

    home.style.display = 'none';
    recent.style.display = 'none';
    games.style.display = 'none';
    favs.style.display = 'none';
    messages.style.display = 'flex';
    activity.style.display = 'none';
    friends.style.display = 'none';
  });

  document.getElementById('activity-btn').addEventListener('click', async function () {
    marker.style.top = '311px';

    home.style.display = 'none';
    recent.style.display = 'none';
    games.style.display = 'none';
    favs.style.display = 'none';
    messages.style.display = 'none';
    activity.style.display = 'flex';
    friends.style.display = 'none';
  });

  document.getElementById('friends-btn').addEventListener('click', async function () {
    marker.style.top = '361px';

    home.style.display = 'none';
    recent.style.display = 'none';
    games.style.display = 'none';
    favs.style.display = 'none';
    messages.style.display = 'none';
    activity.style.display = 'none';
    friends.style.display = 'flex';
  });

  document.getElementById('text').addEventListener('change', async (e) => {
    const appDirPath = await path.appDir();
    fs.writeTextFile(appDirPath + 'storage/cache/user/UserProfile.json', JSON.stringify({ username: e.target.value, }), (err) => {
      if (err) throw err;
    });
  });

  document.getElementById('settings-btn').addEventListener('click', async function () {
    settingsbackblur.style.display = 'flex';
    settings.style.display = 'flex';
  });

  settingsbackblur.addEventListener('click', function () {
    settings.style.display = 'none';
    settingsbackblur.style.display = 'none';
  });

  document.querySelector('.titlebar-settings').addEventListener('click', async () => {
    const appDirPath = await path.appDir();
    const Data = JSON.parse(await fs.readTextFile(appDirPath + 'storage/LauncherData.json'));
    document.querySelectorAll('input[id^=setting-]').forEach((input) => {
      input.checked = Data[input.id.split('-')[1]] ? true : false;
    });
  });

  document.querySelectorAll('input[id^=setting-]').forEach((input) => {
    input.addEventListener('change', async () => {
      const appDirPath = await path.appDir();

      const LauncherData = JSON.parse(await fs.readTextFile(appDirPath + 'storage/LauncherData.json'));
      LauncherData[input.id.split('-')[1]] = document.querySelector(`input[id=${input.id}]`).checked;
      fs.writeTextFile(appDirPath + 'storage/LauncherData.json', JSON.stringify(LauncherData));
    });
  });

  searchbars.item(0).addEventListener('keyup', () => {
    const query = searchbars.item(0).value;
    const allGames = document.querySelectorAll('#allGamesList > div[id^="game-div"]');

    allGames.forEach((game) => {
      if (game.id.split('-').slice(2).join('-').match(new RegExp(`${query}`, 'gi'))) {
        game.style.display = 'block';
      }
      else if (query.length === 0) {
        game.style.display = 'block';
      }
      else {
        game.style.display = 'none';
      }
    });
  });
  searchbars.item(1).addEventListener('keyup', () => {
    const query = searchbars.item(1).value;
    const allGames = document.querySelectorAll('#favGamesList > div[id^="game-div"]');

    allGames.forEach((game) => {
      if (game.id.split('-').slice(2).join('-').match(new RegExp(`${query}`, 'gi'))) {
        game.style.display = 'block';
      }
      else if (query.length === 0) {
        game.style.display = 'block';
      }
      else {
        game.style.display = 'none';
      }
    });
  });
});
</script>
