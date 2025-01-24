<script lang="ts">
    import { onMount } from 'svelte';

    // https://birch.catenarymaps.org/get_realtime_locations/metro~losangeles/metro/0/0

    let isRunning: boolean | null = null;
    let details: string = "";
    let next: string = "";
    let singleTrainTimes: Array<any>= [];

    let indices = {
        'Metro D Line - Wilshire / Western Station': [
            'Union Station',
            'Civic Center / Grand Park',
            'Pershing Square',
            '7th Street / Metro Center',
            'Westlake / MacArthur Park',
            'Wilshire / Vermont',
            'Wilshire / Normandie',
            'Wilshire / Western',
        ],
        'Metro D Line - Union Station': [
            'Wilshire / Western',
            'Wilshire / Normandie',
            'Wilshire / Vermont',
            'Westlake / MacArthur Park',
            '7th Street / Metro Center',
            'Pershing Square',
            'Civic Center / Grand Park',
            'Union Station',
        ],
        'Metro B Line - Union Station': [
            'North Hollywood',
            'Universal City / Studio City',
            'Hollywood / Highland',
            'Hollywood / Vine',
            'Hollywood / Western',
            'Vermont / Sunset',
            'Vermont / Santa Monica',
            'Vermont / Beverly',
            'Wilshire / Vermont',
            'Westlake / MacArthur Park',
            '7th Street / Metro Center',
            'Pershing Square',
            'Civic Center / Grand Park',
            'Union Station',
        ],
        'Metro B Line - North Hollywood Station': [
            'Union Station',
            'Civic Center / Grand Park',
            'Pershing Square',
            '7th Street / Metro Center',
            'Westlake / MacArthur Park',
            'Wilshire / Vermont',
            'Vermont / Beverly',
            'Vermont / Santa Monica',
            'Vermont / Sunset',
            'Hollywood / Western',
            'Hollywood / Vine',
            'Hollywood / Highland',
            'Universal City / Studio City',
            'North Hollywood',
        ],
    }

    async function fetchIsRunning() {

        // initial lookup
        const response = await fetch(
            "https://birch.catenarymaps.org/get_realtime_locations/metro~losangeles/metro/0/0",
        );
        const data = await response.json();
        const hr4ktrips: any = Object.values(data.vehicle_positions).filter((trip: any) => (trip && trip.vehicle && trip.vehicle.id && trip.vehicle.id.startsWith("40") && trip.vehicle.id.length > 4));
        console.log(hr4ktrips);
        if (hr4ktrips.length > 0) {
            // extra train-specific data collection
            const hr4ktrip = hr4ktrips[0]
            console.log(hr4ktrip);
            const tDataResponse = await fetch(`https://birch_req_trip.catenarymaps.org/get_trip_information/metro~losangeles/?trip_id=${hr4ktrip.trip.trip_id}&start_date=${hr4ktrip.trip.start_date}`)
            const tData = await tDataResponse.json();
            console.log("trainset",hr4ktrip.vehicle.id,"",tData)

            // info display

            if(tData && tData.stoptimes)
                tData.stoptimes.map((stoptime:any)=>{
                    console.log(stoptime)
                    const tDate = stoptime?.rt_arrival?.time && new Date(stoptime?.rt_arrival?.time * 1000)

                    if(!!tDate)
                        singleTrainTimes = singleTrainTimes.concat({name: stoptime.name, time: `${tDate.getHours() % 12 || 12}:${tDate.getMinutes()}`})
            })

            console.log(singleTrainTimes)

            isRunning = true;
            details = `HR4000 set ${hr4ktrip.vehicle.id} is running on the ${hr4ktrip.trip.trip_headsign.replace(' - ', ' to ').replace(' Station', '').replace(' Union', ' Union Station')}.`;
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

<div class="wrap">
    <div>
        <h1 style="font-size: 2.5rem;">Is the <b>HR4000</b> running now?</h1>
        {#if isRunning == true}
            <h2 style="font-size:4rem;">Yes!</h2>
            <h3 style="font-size: 2rem;">Go find that goofy steam horn!</h3>
        {:else if isRunning == false}
            <h2 style="font-size:4rem;">No!</h2>
            <h3 style="font-size: 2rem;">Better luck next time :(</h3>
        {:else}
            <h2 style="font-size:4rem;">Loading...</h2>
            <h3 style="font-size: 2rem;">----</h3>
        {/if}
                        {#if isRunning == true}
        <p style="font-size: 1.5rem;">{details}<br />{next}</p>
        

        <h3 style="font-size: 2rem;">Estimated Arrival Times</h3>
        {#each singleTrainTimes as singleTrainTime}
	        <div style="font-size:1.5rem;  padding: 2px 30px;">
		       {singleTrainTime.name} @ {singleTrainTime.time}
	        </div>
        {/each}
        <br/>
        
        {/if}
        <div style="font-size: 1.25rem;color:white;">Built by <a href="https://samuelsharp.com" style="font-size: 1.25rem;color:white;">Samuel Sharp (@42A7C5)</a> and  <a href="https://github.com/ultracodez" style="font-size: 1.25rem;color:white;">Eduardo Maroto (@ultracodez)</a></div>
    </div>
</div>

<style>
    .wrap {
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
    }

    .wrap div {
        text-align: center;
        padding: 30px;

    }
</style>
