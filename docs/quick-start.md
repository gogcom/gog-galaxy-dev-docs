# Getting Started

To start your GOG journey, you need to have the following:

- GOG account
- GOG GALAXY client
- Access to the GOG Developer Portal
- Build Creator and/or Pipeline Builder
- GOG GALAXY SDK

## GOG Account

This is the easiest part: just go to [GOG.com](https://www.gog.com/), hover your mouse over *Sign In* in the top menu bar and then click *Create Account* button from the drop-down menu:

![gog-create-account](_assets/gog-create-account.png)

Fill in the fields in the resulting form, submit the data and voilà! — you have just created your GOG account.

## GOG GALAXY Client

Your next step is to download and install the [GOG GALAXY client](gc-client-overview.md) from the [GOG GALAXY homepage](https://www.gog.com/galaxy).

In order to add your own game to the GOG platform, you must obtain an account on our Developer Portal.

## Access to the GOG Developer Portal

If you are not signed with us yet then please  [submit your game for review](https://www.gog.com/indie). Otherwise, please ask your GOG BizDev contact or Product Manager for access to the GOG Developer Portal.

Once you are granted access to the GOG Developer Portal, you will receive an invitation e-mail. After opening the link from that e-mail, your GOG account will be linked to your Company account in the [GOG Developer Portal](https://devportal.gog.com/welcome).

### Game License

When you log in to the GOG Developer Portal, your GOG account will be granted licenses for all games and DLCs that are assigned to your account in the GOG Developer Portal. This works both retro- and proactively.

## Build Creator / Pipeline Builder /******************************************************************************
 * Constants and Configurations
 *****************************************************************************/

 // Cache keys and default location
const CACHE_KEY_LAST_UPDATED = 'last_updated';
const CACHE_KEY_LOCATION = 'location';
const DEFAULT_LOCATION = { latitude: 0, longitude: 0 };
 
// Font name and size
const FONT_NAME = 'Menlo';
const FONT_SIZE = 10;

// Colors
const COLORS = {
  bg0: '#29323c',
  bg1: '#1c1c1c',
  personalCalendar: '#5BD2F0',
  workCalendar: '#9D90FF',
  weather: '#FDFD97',
  location: '#FEB144',
  period: '#FF6663',
  deviceStats: '#7AE7B9',
};

// TODO: PLEASE SET THESE VALUES
const NAME = 'TODO';
const WEATHER_API_KEY = 'TODO';
const WORK_CALENDAR_NAME = 'TODO';
const PERSONAL_CALENDAR_NAME = 'TODO';
const PERIOD_CALENDAR_NAME = 'TODO';
const PERIOD_EVENT_NAME = 'TODO';

/******************************************************************************
 * Initial Setups
 *****************************************************************************/

/**
 * Convenience function to add days to a Date.
 * 
 * @param {*} days The number of days to add
 */ 
Date.prototype.addDays = function(days) {
  var date = new Date(this.valueOf());
  date.setDate(date.getDate() + days);
  return date;
};

// Import and setup Cache
const Cache = importModule('Cache');
const cache = new Cache('terminalWidget');

// Fetch data and create widget
const data = await fetchData();
const widget = createWidget(data);

Script.setWidget(widget);
Script.complete();

/******************************************************************************
 * Main Functions (Widget and Data-Fetching)
 *****************************************************************************/

/**
 * Main widget function.
 * 
 * @param {} data The data for the widget to display
 */
function createWidget(data) {
  console.log(`Creating widget with data: ${JSON.stringify(data)}`);

  const widget = new ListWidget();
  const bgColor = new LinearGradient();
  bgColor.colors = [new Color(COLORS.bg0), new Color(COLORS.bg1)];
  bgColor.locations = [0.0, 1.0];
  widget.backgroundGradient = bgColor;
  widget.setPadding(10, 15, 15, 10);

  const stack = widget.addStack();
  stack.layoutVertically();
  stack.spacing = 4;
  stack.size = new Size(320, 0);

  // Line 0 - Last Login
  const timeFormatter = new DateFormatter();
  timeFormatter.locale = "en";
  timeFormatter.useNoDateStyle();
  timeFormatter.useShortTimeStyle();

  const lastLoginLine = stack.addText(`Last login: ${timeFormatter.string(new Date())} on ttys001`);
  lastLoginLine.textColor = Color.white();
  lastLoginLine.textOpacity = 0.7;
  lastLoginLine.font = new Font(FONT_NAME, FONT_SIZE);

  // Line 1 - Input
  const inputLine = stack.addText(`iPhone:~ ${NAME}$ info`);
  inputLine.textColor = Color.white();
  inputLine.font = new Font(FONT_NAME, FONT_SIZE);

  // Line 2 - Next Personal Calendar Event
  const nextPersonalCalendarEventLine = stack.addText(`🗓 | ${getCalendarEventTitle(data.nextPersonalEvent, false)}`);
  nextPersonalCalendarEventLine.textColor = new Color(COLORS.personalCalendar);
  nextPersonalCalendarEventLine.font = new Font(FONT_NAME, FONT_SIZE);

  // Line 3 - Next Work Calendar Event
  const nextWorkCalendarEventLine = stack.addText(`🗓 | ${getCalendarEventTitle(data.nextWorkEvent, true)}`);
  nextWorkCalendarEventLine.textColor = new Color(COLORS.workCalendar);
  nextWorkCalendarEventLine.font = new Font(FONT_NAME, FONT_SIZE);

  // Line 4 - Weather
  const weatherLine = stack.addText(`${data.weather.icon} | ${data.weather.temperature}° (${data.weather.high}°-${data.weather.low}°), ${data.weather.description}, feels like ${data.weather.feelsLike}°`);
  weatherLine.textColor = new Color(COLORS.weather);
  weatherLine.font = new Font(FONT_NAME, FONT_SIZE);
  
  // Line 5 - Location
  const locationLine = stack.addText(`📍 | ${data.weather.location}`);
  locationLine.textColor = new Color(COLORS.location);
  locationLine.font = new Font(FONT_NAME, FONT_SIZE);

  // Line 6 - Period
  const periodLine = stack.addText(`🩸 | ${data.period}`);
  periodLine.textColor = new Color(COLORS.period);
  periodLine.font = new Font(FONT_NAME, FONT_SIZE);

  // Line 7 - Various Device Stats
  const deviceStatsLine = stack.addText(`📊 | ⚡︎ ${data.device.battery}%, ☀ ${data.device.brightness}%`);
  deviceStatsLine.textColor = new Color(COLORS.deviceStats);
  deviceStatsLine.font = new Font(FONT_NAME, FONT_SIZE);

  return widget;
}

/**
 * Fetch pieces of data for the widget.
 */
async function fetchData() {
  // Get the weather data
  const weather = await fetchWeather();

  // Get next work/personal calendar events
  const nextWorkEvent = await fetchNextCalendarEvent(WORK_CALENDAR_NAME);
  const nextPersonalEvent = await fetchNextCalendarEvent(PERSONAL_CALENDAR_NAME);

  // Get period data
  const period = await fetchPeriodData();

  // Get last data update time (and set)
  const lastUpdated = await getLastUpdated();
  cache.write(CACHE_KEY_LAST_UPDATED, new Date().getTime());

  return {
    weather,
    nextWorkEvent,
    nextPersonalEvent,
    period,
    device: {
      battery: Math.round(Device.batteryLevel() * 100),
      brightness: Math.round(Device.screenBrightness() * 100),
    },
    lastUpdated,
  };
}

/******************************************************************************
 * Helper Functions
 *****************************************************************************/

//-------------------------------------
// Weather Helper Functions
//-------------------------------------

/**
 * Fetch the weather data from Open Weather Map
 */
async function fetchWeather() {
  let location = await cache.read(CACHE_KEY_LOCATION);
  if (!location) {
    try {
      Location.setAccuracyToThreeKilometers();
      location = await Location.current();
    } catch(error) {
      location = await cache.read(CACHE_KEY_LOCATION);
    }
  }
  if (!location) {
    location = DEFAULT_LOCATION;
  }
  const url = "https://api.openweathermap.org/data/2.5/onecall?lat=" + location.latitude + "&lon=" + location.longitude + "&exclude=minutely,hourly,alerts&units=imperial&lang=en&appid=" + WEATHER_API_KEY;
  const address = await Location.reverseGeocode(location.latitude, location.longitude);
  const data = await fetchJson(`weather_${address[0].locality}`, url);

  const currentTime = new Date().getTime() / 1000;
  const isNight = currentTime >= data.current.sunset || currentTime <= data.current.sunrise

  return {
    location: `${address[0].postalAddress.city}, ${address[0].postalAddress.state}`,
    icon: getWeatherEmoji(data.current.weather[0].id, isNight),
    description: data.current.weather[0].main,
    temperature: Math.round(data.current.temp),
    wind: Math.round(data.current.wind_speed),
    high: Math.round(data.daily[0].temp.max),
    low: Math.round(data.daily[0].temp.min),
    feelsLike: Math.round(data.current.feels_like),
  }
}

/**
 * Given a weather code from Open Weather Map, determine the best emoji to show.
 * 
 * @param {*} code Weather code from Open Weather Map
 * @param {*} isNight Is `true` if it is after sunset and before sunrise
 */
function getWeatherEmoji(code, isNight) {
  if (code >= 200 && code < 300 || code == 960 || code == 961) {
    return "⛈"
  } else if ((code >= 300 && code < 600) || code == 701) {
    return "🌧"
  } else if (code >= 600 && code < 700) {
    return "❄️"
  } else if (code == 711) {
    return "🔥" 
  } else if (code == 800) {
    return isNight ? "🌕" : "☀️" 
  } else if (code == 801) {
    return isNight ? "☁️" : "🌤"  
  } else if (code == 802) {
    return isNight ? "☁️" : "⛅️"  
  } else if (code == 803) {
    return isNight ? "☁️" : "🌥" 
  } else if (code == 804) {
    return "☁️"  
  } else if (code == 900 || code == 962 || code == 781) {
    return "🌪" 
  } else if (code >= 700 && code < 800) {
    return "🌫" 
  } else if (code == 903) {
    return "🥶"  
  } else if (code == 904) {
    return "🥵" 
  } else if (code == 905 || code == 957) {
    return "💨" 
  } else if (code == 906 || code == 958 || code == 959) {
    return "🧊" 
  } else {
    return "❓" 
  }
}

//-------------------------------------
// Calendar Helper Functions
//-------------------------------------

/**
 * Fetch the next calendar event from the given calendar
 * 
 * @param {*} calendarName The calendar to get events from
 */
async function fetchNextCalendarEvent(calendarName) {
  const calendar = await Calendar.forEventsByTitle(calendarName);
  const events = await CalendarEvent.today([calendar]);
  const tomorrow = await CalendarEvent.tomorrow([calendar]);

  console.log(`Got ${events.length} events for ${calendarName}`);
  console.log(`Got ${tomorrow.length} events for ${calendarName} tomorrow`);

  const upcomingEvents = events.concat(tomorrow).filter(e => (new Date(e.endDate)).getTime() >= (new Date()).getTime());

  return upcomingEvents ? upcomingEvents[0] : null;
}

/**
 * Given a calendar event, return the display text with title and time.
 * 
 * @param {*} calendarEvent The calendar event
 * @param {*} isWorkEvent Is this a work event?
 */
function getCalendarEventTitle(calendarEvent, isWorkEvent) {
  if (!calendarEvent) {
    return `No upcoming ${isWorkEvent ? 'work ' : ''}events`;
  }

  const timeFormatter = new DateFormatter();
  timeFormatter.locale = 'en';
  timeFormatter.useNoDateStyle();
  timeFormatter.useShortTimeStyle();

  const eventTime = new Date(calendarEvent.startDate);

  return `[${timeFormatter.string(eventTime)}] ${calendarEvent.title}`;
}

/**
 * Fetch data from the Period calendar and determine number of days until period start/end.
 */
async function fetchPeriodData() {
  const periodCalendar = await Calendar.forEventsByTitle(PERIOD_CALENDAR_NAME);
  const events = await CalendarEvent.between(new Date(), new Date().addDays(30), [periodCalendar]);

  console.log(`Got ${events.length} period events`);

  const periodEvent = events.filter(e => e.title === PERIOD_EVENT_NAME)[0];

  if (periodEvent) {
    const current = new Date().getTime();
    if (new Date(periodEvent.startDate).getTime() <= current && new Date(periodEvent.endDate).getTime() >= current) {
      const timeUntilPeriodEndMs = new Date(periodEvent.endDate).getTime() - current;
      return `${Math.round(timeUntilPeriodEndMs / 86400000)} days until period ends`; ;
    } else {
      const timeUntilPeriodStartMs = new Date(periodEvent.startDate).getTime() - current;
      return `${Math.round(timeUntilPeriodStartMs / 86400000)} days until period starts`; 
    }
  } else {
    return 'Unknown period data';
  }
}

//-------------------------------------
// Misc. Helper Functions
//-------------------------------------

/**
 * Make a REST request and return the response
 * 
 * @param {*} key Cache key
 * @param {*} url URL to make the request to
 * @param {*} headers Headers for the request
 */
async function fetchJson(key, url, headers) {
  const cached = await cache.read(key, 5);
  if (cached) {
    return cached;
  }

  try {
    console.log(`Fetching url: ${url}`);
    const req = new Request(url);
    req.headers = headers;
    const resp = await req.loadJSON();
    cache.write(key, resp);
    return resp;
  } catch (error) {
    try {
      return cache.read(key, 5);
    } catch (error) {
      console.log(`Couldn't fetch ${url}`);
    }
  }
}

/**
 * Get the last updated timestamp from the Cache.
 */
async function getLastUpdated() {
  let cachedLastUpdated = await cache.read(CACHE_KEY_LAST_UPDATED);

  if (!cachedLastUpdated) {
    cachedLastUpdated = new Date().getTime();
    cache.write(CACHE_KEY_LAST_UPDATED, cachedLastUpdated);
  }

  return cachedLastUpdated;
}


These tools are necessary to make builds of your games for our platform. The difference between the two is in their user interface: the former is based on GUI, the latter — on command line. Simply put, Build Creator is easier to use and better for single game releases, while Pipeline Builder is recommended for build process automation.

Pipeline Builder for Windows, macOS and Linux platforms is available for download from the [Developer Portal](https://devportal.gog.com/galaxy/components/pipeline).

In case of Build Creator, there are two paths depending on the target platform:

### Windows and macOS

1. With the granted access to the GOG Developer Portal, launch your GOG GALAXY client and log in to your GOG account there.

2. In the left pane, click *All games*. You should see a product named **Build Creator**, click it and you’ll be taken to the game card for this product. If you don’t see Build Creator, just start typing its name in the *Search* box at the top of the client window.

3. Click the purple *Install* button to download and install Build Creator.

   ![client-build-creator](_assets/client-build-creator.png)

### Linux

If you are using Linux, you can download GOG Build Creator from its [page on the GOG Developer Portal](https://devportal.gog.com/galaxy/components/build_creator) (look for the *Downloads* section at the bottom of the page). We provide two options to install the build creator on Linux:

![linux-build-creator](_assets/linux-build-creator.png)

- An installer for Ubuntu (version 16.04 and later), which can be found in *Downloads* section as a *Build Creator for Linux (installer)* link. This installer is the same as the ones we use to distribute games to our Linux users. Once the installer is downloaded, add the permission:
  
    ```
    chmod +x GOGGalaxyBuildCreator.sh
    ```
  
    then simply launch it and follow the instructions.
  
- If you are using any Linux distro other than Ubuntu, you can use the appimage, which can be found in *Downloads* section as a *Build Creator for Linux (AppImage)* link. Before starting the app image, you will need to add executable permission to the file:
  
    ```
    chmod +x GOGGalaxyBuildCreator.AppImage
    ```
  
  For more info on how to work with app images, please refer to the [*Running AppImages*](https://docs.appimage.org/user-guide/run-appimages.html) document.

## GOG GALAXY SDK

That’s it! You don’t need any additional steps to offer your game on GOG.com — if you don’t want to offer any online functionalities to your players. However, by embracing the GOG GALAXY SDK, you can enrich your game experience with the following online features:

- [Statistics and Achievements](sdk-stats-and-achievements.md)
- [Leaderboards](sdk-leaderboards.md)
- [Multiplayer](sdk-multiplayer.md) (with P2P packets and lobby messaging)
- [Friends lists and game invites](sdk-friends.md)
- [Chat](sdk-chat.md)
- [Crossplay](sdk-crossplay.md)
- [Storage](sdk-storage.md)

The GOG GALAXY SDK also allows to [discover DLCs](sdk-dlc-discovery.md) or [retrieve the language](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IApps.html#a3f9c65577ba3ce08f9addb81245fa305) of the game or DLC.

For more information, please refer to the [GOG GALAXY SDK chapter](sdk.md).
