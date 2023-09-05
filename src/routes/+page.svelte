<script>
// @ts-nocheck

	import { tick } from "svelte";

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
  $: action = "";
  let timerSet = [];
  let displayTime = "00:00:00";
  let log = [];
  let loot = new Map();
  let choices = [];
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
      'percent': 1,
      'id': 0,
      'time': 2000,
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
      'time': 200,
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
      'time': 1000,
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
      'time': 2000,
      'slug': 'titanium',
      'limit': 1,
      'disabled': false,
    },
    {
      'name': "Delicate machinery",
      'chance': 0.1,
      'number': 2,
      'percent': 1,
      'id': 4,
      'time': 2000,
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
  */
  const delay = (ms) => {
    return new Promise(resolve => {
      setTimeout(resolve, ms);
    });
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
    addToChoices(index);
    // if(loot.length > 0) {
    //   loot = [];
    // }
  }

  /**
   * handleRemoveButtonClick
   * @param id - id of the object in the array
   */
  const handleRemoveButtonClick = (id) => {
    const optionsIndex = options.findIndex((item) => item.id === id);
    const choicesIndex = choices.findIndex((item) => item.id === id);
    options[optionsIndex].disabled = false;
    limit -= options[optionsIndex].time;
    removeFromChoices(choicesIndex);
  }

  const handleLoot = (item) => {
    if(loot.has(item.slug)) {
      let temp = loot.get(item.slug);
      console.log('temp: ', temp);
      temp.number += item.number;
      loot.set(item.slug, temp);
    } else {
      loot.set(item.slug, {
        'name': item.name,
        'slug': item.slug,
        'number': item.number,
      });
    }
  }

  /**
   * handleExecuteButton
   */
   const handleExecuteButton = async() => {
    // loot = [];
    log = [];

    if(choices.length > 0) {
      running = true;
      // startTimer(limit);
      let temp = choices.slice();
      for (const item of temp) {
        action = "";
        await delay(item.time);
        let roll = getRandomInt(1, 100);
        const optionsIndex = options.findIndex((option) => option.id === item.id);
        console.log(`rolled ${roll}, needed ${item.chance * 100}`);
        if (roll <= (Math.round(item.chance * 100))) {
          handleLoot(item)
          // loot.push(item);
          console.log("successful loot", loot);
          console.log('array of loot', Array.from(loot));
          log.push({'name': item.name, 'message': `successful loot ${item.name}`});
          item.action = true;
        } else { 
          item.action = false;
          log.push({'name': item.name, 'message': `missed loot ${item.name}`});
          console.log("no loot", loot);
        }
        action = action;
        // removeFromChoices(0);
        console.log("options index", optionsIndex);
        // options[optionsIndex].disabled = false;
        loot = loot;
        log = log;
      }

      running = false;
      options = options;
      loot = loot;

      // setTimeout(() => {
      //   choices.forEach(choice => {
      //     let roll = getRandomInt(1, 100);
      //     const optionsIndex = options.findIndex((item) => item.id === choice.id);
      //     console.log('roll', roll);
      //     console.log('chance', choice.chance);
      //     if (roll <= (Math.round(choice.chance * 100))) {
      //       loot.push(choice);
      //     }
      //   // options[optionsIndex].disabled = false;
      //     running = false;
      //   });

      //   loot = loot;
      //   // choices = [];
      //   // console.log("OKAY");
      //   // console.log("Tick through the array one by one");
      //   // console.log("Roll the chance to see if successful");
      //   console.log("Need to determine rewards");
      //   console.log("Ideally this would happen in sequence and on a timer");
      //   console.log("limit to some arbitrary time, 45mins?");
      // }, limit);
    } else {
      error = "No choices to execute";
    }
  }

  /**
   * addToChoices
   * @param i - index of the object in the array
   */
  const addToChoices = (i) => {
    choices.push(options[i]);
    // this makes it reactive
    choices = choices;
    options = options;
  }

  /**
   * removeFromChoices
   * @param i - index of the object in the array
   */
  const removeFromChoices = (i) => {
    choices.splice(i, 1);
    // this makes it reactive
    choices = choices;
    options = options;
  }
</script>

<h1>Welcome to ScrapKit</h1>
<p>Choose some options that you want to execute.</p>
<section>
  <ul>
    <li>Options</li>
    {#each options as item (item.id)}
      <li>
        <button class="list-button" disabled={item.disabled || running} on:click={ () => handleAddButtonClick(item.id) }>{item.name} {Math.round(item.chance * 100)}% - {item.time / 1000}s</button>
      </li>
    {/each}
  </ul>

  <ul>
    <li>Choices</li>
    {#each choices as item (item.id)}
      <li>
        <button class="list-button" disabled={running} on:click={ () => handleRemoveButtonClick(item.id) } >{item.name} {Math.round(item.chance * 100)}% - {item.time / 1000}s</button>
      </li>
    {/each}
  </ul>
</section>
<section>
  <button on:click={handleExecuteButton}>Execute</button>
  {#if error}
    <p>{error}</p>
  {/if}
</section>
<section>
  <ul>
    <li>Log:</li>
    {#each log as item}
      <li>{item.message}</li>
    {/each}
  </ul>
</section>
<section>
  <!-- <p>Timer: {displayTime}</p> -->
  <ul>
    <li>Loot:</li>
    {#each loot as item}
      <li>{item[0]}: {item[1].number}</li>
    {/each}
  </ul>
</section>

<style>
  h1,
  p {
    text-align: center;
  }

  section {
    column-gap: 20px;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    margin: auto;
    max-width: 800px;
    width: 100%;
  }

  ul {
    list-style: none;
    padding: 0;
    max-width: 50%;
    width: 100%;
  }

  .list-button {
    display: block;
    margin: 10px 0;
    width: 100%;
  }
</style>
