<script>
// @ts-nocheck

  let limit = 0;
  let error = null;
  let running = false;
  let timerInterval;
  let timer = {
    total: 0,
    tens: 0,
    seconds: 0,
    minutes: 0
  };
  let timerSet = [];
  let now = Date.now();
  let currentTimer;
  let remainingTime;
  let displayTime = "00:00:00";
  let log = [];
  let loot = new Map();
  let queue = [];
  let options = [
    /**
     * name: display title
     * chance: roll needed to acquire the item (maybe this should be calculated based on percent and number)
     * percent: percent of the item able to acquire on a successful roll
     * id: unique id of the item
     * time: time it takes to complete the action
     * limit: in house ship limit
     * slug: use to stack inventory, items that share slugs should combine
     * disabled: whether the button is disabled
    */
    {
      'name': "Scrap the system",
      'chance': 0.58,
      'number': 1,
      'percent': 0.58,
      'id': 0,
      'time': 5000,
      'slug': 'sys01',
      'limit': 1,
      'disabled': false,
    },
    {
      'name': "Small Windows",
      'chance': 0.90,
      'number': 2,
      'percent': 1,
      'id': 1,
      'time': 1000,
      'slug': 'sm_windows',
      'limit': 2,
      'disabled': false,
    },
    {
      'name': "Large Windows",
      'chance': 0.90,
      'number': 2,
      'percent': 1,
      'id': 2,
      'time': 5000,
      'slug': 'lg_windows',
      'limit': 1,
      'disabled': false,
    },
    {
      'name': "Work on the exterior",
      'chance': 0.95,
      'number': 200,
      'percent': 0.95,
      'id': 3,
      'time': 5000,
      'slug': 'titanium',
      'limit': 10,
      'disabled': false,
    },
    {
      'name': "Delicate machinery",
      'chance': 0.1,
      'number': 2,
      'percent': 1,
      'id': 4,
      'time': 5000,
      'slug': 'sys02',
      'limit': 1,
      'disabled': false,
    },
  ]


  const startTimer = (start = 4500) => {
    timer.total = 0;
    timer.tens = 0;
    timer.seconds = 0;
    timer.minutes = 0;
    displayTime = "00:00:00";

    timerInterval = setInterval(() => tickTimer(start), 10);
  }

  const stopTimer = (interval = timerInterval) => {
    clearInterval(interval);
  }

  const tickTimer = (start, factor = 1) => {
    timer.total += (10 * factor)
    if(timer.total <= start) {
      timer.tens += 1
      if (timer.tens > 99) {
        timer.tens = 0
        timer.seconds += 1
      }
  
      if (timer.seconds === 60) {
        timer.seconds = 0
        timer.minutes += 1
      }

      // timer.tens = (((timer.total % 60000) % 1000) / 10);
      // timer.seconds = Math.floor((timer.total % 60000) / 1000);
      // timer.minutes = Math.floor(timer.total / 60000);
      
  
      displayTime = `${timer.minutes < 10 ? "0" + timer.minutes: timer.minutes}:${timer.seconds < 10 ? "0" + timer.seconds: timer.seconds}:${timer.tens < 10 ? "0" + timer.tens: timer.tens}`;
    } else {
      console.log(timer);
      stopTimer(timerInterval);
    }
  }

  /**
   * delay
   * @param ms - delay to resolve the timeout
  */
  const delay = (ms) => {
    now = Date.now();

    return new Promise(resolve => {
      timerInterval = setTimeout(resolve, ms);
    });
  }

  /**
   * pauseTimer
  */
  const pauseTimer = () => {
    clearTimeout(timerInterval);
    remainingTime = currentTimer (now - Date.now());
    // while loop
    // call async function that and await for the next call
  }

  /**
   * resumeTimer
  */
  const resumeTimer = () => {
    delay(remainingTime);
  }

  /**
   * getRandomInt
   * @param min - eg 1
   * @param max - eg 100
   */
  const getRandomInt = (min, max) => {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1) + min);
  }

  /**
   * handleAddButtonClick
   * @param id - id of the object in the array
   */
  const handleAddButtonClick = (id) => {
    const index = options.findIndex((item) => item.id === id);
    options[index].disabled = true;
    limit += options[index].time;
    addToQueue(index);
  }

  /**
   * handleQueueButtonClick
   * @param id - id of the object in the array
   */
  const handleQueueButtonClick = (id) => {
    const optionsIndex = options.findIndex((item) => item.id === id);
    const queueIndex = queue.findIndex((item) => item.id === id);
    options[optionsIndex].disabled = false;
    limit -= options[optionsIndex].time;
    removeFromChoices(queueIndex);
  }

  /**
   * handleLog
   * @param msg - add msg to the log, if max size is hit, remove the last item.
   * Items should be prepended to the array
   */
  const handleLog = (msg) => {
    console.log(msg);
    if(log.length === 5) {
      log.pop();
    }
    log.unshift(msg);
    log = log;
  }

  /**
   * handleInventory
   * handle the looting of an item: check if it exists in the map,
   * if it does override with new count. Otherwise just add it to the map.
   * @param item - lootable object
   */
  const handleInventory = (item) => {
    if(loot.has(item.slug)) {
      let temp = loot.get(item.slug);
      console.log('temp: ', temp);
      temp.number += Math.round(item.number * item.percent);
      loot.set(item.slug, temp);
    } else {
      loot.set(item.slug, {
        'name': item.name,
        'slug': item.slug,
        'number': Math.round(item.number * item.percent),
      });
    }
  }

  const handleLoot = (item) => {
    let roll = getRandomInt(1, 100);
    console.log(`rolled ${roll}, needed ${item.chance * 100}`);
    return roll < Math.round(item.chance * 100);
  }

  /**
   * handleExecuteButton
   */
   const handleExecuteButton = async() => {
    // loot = [];

    if(queue.length > 0) {
      running = true;
      // startTimer(limit);
      let temp = queue.slice();
      for (const item of temp) {
        currentTimer = item.time;
        await delay(item.time);
        const optionsIndex = options.findIndex((option) => option.id === item.id);
        let isLooted = handleLoot(item);
        if (isLooted) {
          handleInventory(item)
          handleLog(`successful loot ${item.name}`)
          item.action = true;
        } else { 
          item.action = false;
          handleLog(`missed loot ${item.name}`);
        }
        // removeFromChoices(0);
        console.log("options index", optionsIndex);
        // options[optionsIndex].disabled = false;
        loot = loot;
      }

      running = false;
      options = options;
      loot = loot;
    } else {
      error = "No queue to execute";
    }
  }

  /**
   * addToQueue
   * @param i - index of the object in the array
   */
  const addToQueue = (i) => {
    queue.push(options[i]);
    // this makes it reactive
    queue = queue;
    options = options;
  }

  /**
   * removeFromChoices
   * @param i - index of the object in the array
   */
  const removeFromChoices = (i) => {
    queue.splice(i, 1);
    // this makes it reactive
    queue = queue;
    options = options;
  }
</script>

<main class="gap-4 grid grid-cols-12 max-w-7xl m-auto">
  <h1 class="text-3xl text-center font-bold col-span-full">Welcome to ScrapKit</h1>
  <p class="text-center col-span-full">Choose some options that you want to execute.</p>
  <ul class="col-span-4 inline-block">
    <li class="font-bold">Options</li>
    {#each options as item (item.id)}
      <li>
        <button class="list-button bg-indigo-400 rounded-sm text-white text-lg p-2 hover:bg-indigo-600 disabled:bg-gray-800" disabled={item.disabled || running} on:click={ () => handleAddButtonClick(item.id) }>{item.name} {Math.round(item.chance * 100)}% - {item.time / 1000}s</button>
      </li>
    {/each}
  </ul>

  <ul class="col-span-4 inline-block">
    <li class="font-bold">Queue</li>
    {#each queue as item (item.id)}
      <li>
        <button class="list-button bg-green-300 rounded-sm text-white text-lg p-2 hover:bg-green-500" disabled={running} on:click={ () => handleQueueButtonClick(item.id) } >{item.name} {Math.round(item.chance * 100)}% - {item.time / 1000}s</button>
      </li>
    {/each}
  </ul>
  <ul class="col-span-4 inline-block">
    <li>Loot:</li>
    {#each loot as item}
      <li>{item[0]}: {item[1].number}</li>
    {/each}
  </ul>
  <button class="col-start-1 bg-green-300 rounded-sm text-white text-lg p-2 hover:bg-green-500" on:click={handleExecuteButton}>Execute</button>
  {#if error}
    <p>{error}</p>
  {/if}
  <ul class="col-start-1 col-span-4 inline-block">
    <li>Log:</li>
    {#each log as item}
      <li>{item}</li>
    {/each}
  </ul>
</main>

<style global lang="postcss">
  ul {
    list-style: none;
    padding: 0;
    width: 100%;
  }

  .list-button {
    display: block;
    margin: 10px 0;
    width: 100%;
  }
</style>
