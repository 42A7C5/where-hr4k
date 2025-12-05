<script lang="ts">
  import "../app.css";

  import { onMount } from "svelte";

  // https://birch.catenarymaps.org/get_realtime_locations/metro~losangeles/metro/0/0

  let isRunning: boolean | null = null;
  let details: string = "";
  let next: string = "";
  let singleTrainTimes: Array<any> = [];
  const isDev = false;

  let indices = {
    "Metro D Line - Wilshire / Western Station": [
      "Union Station",
      "Civic Center / Grand Park",
      "Pershing Square",
      "7th Street / Metro Center",
      "Westlake / MacArthur Park",
      "Wilshire / Vermont",
      "Wilshire / Normandie",
      "Wilshire / Western",
    ],
    "Metro D Line - Union Station": [
      "Wilshire / Western",
      "Wilshire / Normandie",
      "Wilshire / Vermont",
      "Westlake / MacArthur Park",
      "7th Street / Metro Center",
      "Pershing Square",
      "Civic Center / Grand Park",
      "Union Station",
    ],
    "Metro B Line - Union Station": [
      "North Hollywood",
      "Universal City / Studio City",
      "Hollywood / Highland",
      "Hollywood / Vine",
      "Hollywood / Western",
      "Vermont / Sunset",
      "Vermont / Santa Monica",
      "Vermont / Beverly",
      "Wilshire / Vermont",
      "Westlake / MacArthur Park",
      "7th Street / Metro Center",
      "Pershing Square",
      "Civic Center / Grand Park",
      "Union Station",
    ],
    "Metro B Line - North Hollywood Station": [
      "Union Station",
      "Civic Center / Grand Park",
      "Pershing Square",
      "7th Street / Metro Center",
      "Westlake / MacArthur Park",
      "Wilshire / Vermont",
      "Vermont / Beverly",
      "Vermont / Santa Monica",
      "Vermont / Sunset",
      "Hollywood / Western",
      "Hollywood / Vine",
      "Hollywood / Highland",
      "Universal City / Studio City",
      "North Hollywood",
    ],
  };

  async function fetchIsRunning() {
    // initial lookup
    const response = await fetch(
      "https://birch.catenarymaps.org/get_realtime_locations/metro~losangeles/metro/0/0"
    );
    const data = await response.json();
    const hr4ktrips: any = Object.values(data.vehicle_positions).filter(
      (trip: any) =>
        trip &&
        trip.vehicle &&
        trip.vehicle.id &&
        trip.vehicle.id.startsWith(isDev ? "5" : "40") &&
        trip.vehicle.id.length > 4
    );
    console.log(hr4ktrips);
    if (hr4ktrips.length > 0) {
      // extra train-specific data collection
      const hr4ktrip = hr4ktrips[0];
      console.log(hr4ktrip);
      const tDataResponse = await fetch(
        `https://birch_req_trip.catenarymaps.org/get_trip_information/metro~losangeles/?trip_id=${hr4ktrip.trip.trip_id}&start_date=${hr4ktrip.trip.start_date}`
      );
      const tData = await tDataResponse.json();
      console.log("trainset", hr4ktrip.vehicle.id, "", tData);

      // info display

      if (tData && tData.stoptimes)
        tData.stoptimes.map((stoptime: any) => {
          console.log(stoptime);
          const tDate =
            stoptime?.rt_arrival?.time &&
            new Date(stoptime?.rt_arrival?.time * 1000);

          // const formattedMinutes = tDate.getMinutes();

          // const tMinutes = tDate.getMinutes();

          if (!!tDate)
            singleTrainTimes = singleTrainTimes.concat({
              name: stoptime.name,
              time: `${tDate.getHours() % 12 || 12}:${`${tDate.getMinutes() < 10 ? "0" : ""}${tDate.getMinutes()}`}`,
            });
        });

      console.log(singleTrainTimes);

      isRunning = true;
      details = `HR4000 set ${hr4ktrip.vehicle.id} is running on the ${hr4ktrip.trip.trip_headsign.replace(" - ", " to ").replace(" Station", "").replace(" Union", " Union Station")}.`;
      // @ts-ignore
      next = `> ${indices[hr4ktrip.trip.trip_headsign][hr4ktrip.current_stop_sequence]} is next`;
    } else {
      isRunning = false;
    }
  }

  onMount(() => {
    fetchIsRunning();
  });
</script>

<div class="h-screen flex px-10 lg:px-0">
  <div class="mx-auto text-center pt-32 md:pt-32">
    <h1 class="text-2xl md:text-4xl">Is the <b>HR4000</b> running now?</h1>
    {#if isRunning == true}
      <h2 class="text-4xl md:text-6xl pt-6">Yes!</h2>
      <h3 class="text-xl md:text-3xl pt-6">Go find that goofy steam horn!</h3>
    {:else if isRunning == false}
      <h2 class="text-4xl md:text-6xl pt-6">No!</h2>
      <h3 class="text-xl md:text-3xl pt-6">Better luck next time :(</h3>
    {:else}
      <h2 class="text-4xl md:text-6xl pt-6">Loading...</h2>
      <h3 class="text-xl md:text-3xl pt-6">----</h3>
    {/if}
    {#if isRunning == true}
      <p class="text-md md:text-2xl pt-4">{details}</p>
      <p class="text-lg md:text-2xl font-semibold pt-2">{next}</p>

      <h3 class="text-xl md:text-3xl pt-8 pb-2 underline">
        Estimated Arrival Times
      </h3>
      {#each singleTrainTimes as singleTrainTime}
        <div class="text-md md:text-2xl">
          {singleTrainTime.name} @
          <span class="font-semibold">{singleTrainTime.time}</span>
        </div>
      {/each}
      <br />
    {/if}
    <div class="text-sm md:text-xl pt-4 pb-12">
      Built by <a class="underline" href="https://samuelsharp.com"
        >Samuel Sharp (@42A7C5)</a
      >
      and
      <a class="underline" href="https://github.com/ultracodez"
        >Eduardo Maroto (@ultracodez)</a
      >
    </div>
  </div>
</div>
