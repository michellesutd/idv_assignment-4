<script>
  import { scaleLinear, scaleBand, scaleTime } from "d3-scale";
  import { data } from "./data.js";
  import {
    Graphic,
    Section,
    PointLayer,
    RectangleLayer,
    Line,
    XAxis,
    YAxis
  } from "@snlab/florence";

  import SvelteTooltip from 'svelte-tooltip';
  // import { BarChart }  from 'd3node-barchart';
  import DataContainer from "@snlab/florence-datacontainer";

  let chartData = data;

  let sales = new DataContainer(chartData);

  // overall settings
  const padding = { left: 90, bottom: 30, top: 0, right: 10 };
  const color = "rgb(93, 134, 156)";
  const color1 = "rgb(187, 46, 46)";
  console.log("aaaaaaaaaaaaaaaaaa",sales)
  // data for facet 1
  let salesPerType = sales
    .groupBy("flat_type")
    .summarise({ total_count: { resale_price: "count" } });
  // data for facet 2
  let salesDistribution = sales
    .bin({ groupBy: "resale_price", method: "EqualInterval", numClasses: 20 })
    .summarise({ total_count: { resale_price: "count" } });
  // data for facet 4
  let salesOverTime = sales
    .groupBy("month")
    .summarise({ mean_price: { resale_price: "mean" } })
    .arrange({ month: "ascending" }) // sort by month (otherwise line will criss-cross)
    .mutate({ month_as_date: row => new Date(row.month) }); // convert to proper date so we can use date scale


  //data for facet 6

  let salesplot = sales
    .groupBy("storey_range")
    .summarise({ total_count: { resale_price: "count" } });
  
   // data for facet 7
  let salesOverFloaType = sales
    // .bin({ groupBy: "flat_type", numClasses: 20 })
    .groupBy("flat_type")
    .summarise({ mean_price: { resale_price: "mean" } });

  console.log(salesOverFloaType);
  // data for facet 8
  let salesflat_model = sales
    .groupBy("flat_model")
    .summarise({ total_count: { resale_price: "count" } });

  let towns = [];
  for (let i = 0; i < data.length; i++) {
    let element = data[i];
    let f = false;
    for(let j = 0; j< towns.length ; j++){
      if(towns[j] == element.town)
        {
          // console.log("town name",towns[j]);
          f = true;
          break;
        }
    }
    if (!f) {
      towns.push(element.town);
    }
  }

  function selTownWithName(townName) {
    let Towndata = [];
    if(townName == 'ALL TOWN'){
      Towndata = data;
      return Towndata;
    }

    for (let i = 0; i < data.length; i++) {
      let element = data[i];
      if (element.town == townName) {
        Towndata.push(element);
      }
    }
   
    return Towndata; 
  }

  
  function selTown(e) {

    let TownchartData = selTownWithName(e.target.value);
    sales = new DataContainer(TownchartData);
        
  // data for facet 1
  salesPerType = sales
    .groupBy("flat_type")
    .summarise({ total_count: { resale_price: "count" } });
  // data for facet 8
  salesflat_model = sales
    .groupBy("flat_model")
    .summarise({ total_count: { resale_price: "count" } });
  // data for facet 2
   salesDistribution = sales
    .bin({ groupBy: "resale_price", method: "EqualInterval", numClasses: 20 })
    .summarise({ total_count: { resale_price: "count" } });

  // data for facet 4
   salesOverTime = sales
    .groupBy("month")
    .summarise({ mean_price: { resale_price: "mean" } })
    .arrange({ month: "ascending" }) // sort by month (otherwise line will criss-cross)
    .mutate({ month_as_date: row => new Date(row.month) }); // convert to proper date so we can use date scale

  //data for facet
  salesplot = sales
    .groupBy("storey_range")
    .summarise({ total_count: { resale_price: "count" } });
  
  // data for facet 7
  salesOverFloaType = sales
    // .bin({ groupBy: "flat_type", numClasses: 20 })
    .groupBy("flat_type")
    .summarise({ mean_price: { resale_price: "mean" } });


  console.log(salesOverTime);
  // data for facet 7
  
  }


  // show info data
  let showinfo = 0;
  function ShowData(e){
    showinfo =Math.round( salesOverFloaType._data.mean_price[e.key]);
  }
</script>

<style>
  .header{
    border-bottom: 1px #080808 solid;
    padding: 2px;
  }
  .choose-select label{
    font-size: 20px;
  }
  .main-chart {
    padding: 20px;
  }
  .header-title{
    text-align: center;
    /* margin-top: -70px; */
  }
  .graph-chart{
    display: inline-block;
  }
  .graph-chart > p{
    padding-left: 35px;
    padding-bottom: 15px;
  }
  
</style>

<div class="graph">
  <div class="header">
    
    <div class = "header-title">
      <h1>HDB Resale Flat Prices Data</h1>
      <label>This website presents data on HDB resale flat prices from January 2017</label>
    </div>
    <div class="choose-select">
      <label>Choose a town :</label>
      <select on:change={e => { selTown(e); }}>
      <option>ALL TOWN</option>
        {#if towns.length === 0}
          <option>No towns</option>
        {:else}
          {#each towns as town}
            <option value={town}>{town}</option>
          {/each}
        {/if}
      </select>
    </div>
  </div>

  <div class="main-chart">
    <!-- main chart -->
    <!-- 1 -->
    <div class="graph-chart">
      <p> 1.Count of Units by Flat Type </p>
      <Graphic width={425} height={425}>
        <!-- 1 -->

        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleLinear().domain([
            0,
            salesPerType.domain('total_count')[1]
          ])}
          scaleY={scaleBand()
            .domain(salesPerType.domain('flat_type'))
            .padding(0.5)}>
          <RectangleLayer
            x1={0}
            x2={salesPerType.column('total_count')}
            y1={salesPerType.column('flat_type')}
            y2={({ scaleY }) => salesPerType.map('flat_type', ft => scaleY(ft) + scaleY.bandwidth())}
            fill={color} />
          <XAxis labelFontSize={8} title="Count" />
          <YAxis labelFontSize={8} title="Flat Type" />
        </Section>
      </Graphic>
    </div>
    <!-- 2 -->
    <div class="graph-chart">
      <p> 2.Count of Units by Resale Price </p>
      <Graphic width={425} height={425}>
        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleLinear().domain(salesDistribution.domain('bins'))}
          scaleY={scaleLinear().domain([
            0,
            salesDistribution.domain('total_count')[1]
          ])}>
          <RectangleLayer
            x1={salesDistribution.map('bins', bin => bin[0])}
            x2={salesDistribution.map('bins', bin => bin[1])}
            y1={0}
            y2={salesDistribution.column('total_count')}
            fill={color} />
          <XAxis tickCount={5} labelFontSize={8} title="Resale Price" />
          <YAxis labelFontSize={8} title="Count" />
        </Section>
      </Graphic>
    </div>
    <!-- 3 -->
    <div class="graph-chart">
      <p> 3.Mean Resale Price Over Time </p>
      <Graphic width={425} height={425}>
        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleTime().domain(salesOverTime.domain('month_as_date'))}
          scaleY={scaleLinear().domain(salesOverTime.domain('mean_price'))}>
          <Line
            x={salesOverTime.column('month_as_date')}
            y={salesOverTime.column('mean_price')}
            stroke={color}
            strokeWidth={2} />
          <XAxis tickCount={3} labelFontSize={8} title="Date" />
          <YAxis labelFontSize={8} title="Mean Resale Price" />
        </Section>
      </Graphic>
    </div>
    <!-- 4 -->
    <div class="graph-chart">
      <p> 4.Resale Price vs Floor Area(sqm)</p>
      <Graphic width={400} height={425}>
        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleLinear().domain(sales.domain('floor_area_sqm'))}
          scaleY={scaleLinear().domain(sales.domain('resale_price'))}>
          <PointLayer
            x={sales.column('floor_area_sqm')}
            y={sales.column('resale_price')}
            fill={color}
            opacity={0.5} />
          <!-- <Scatter /> -->
          <XAxis tickCount={6} labelFontSize={8} title="Floor Area (sqm)" />
          <YAxis labelFontSize={8} title="Resale Price" />
        </Section>
      </Graphic>
    </div>
    <!-- 5 -->
    <div class="graph-chart">
      <p> 5.Resale Price(Years) vs Remaining Lease</p>
      <Graphic width={425} height={425}>
        <!-- 5 -->
        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleLinear().domain(sales.domain('resale_price'))}
          scaleY={scaleLinear().domain(sales.domain('remaining_lease'))}>
          <PointLayer
            x={sales.column('resale_price')}
            y={sales.column('remaining_lease')}
            fill={color1}
            opacity={0.5} />
          <XAxis tickCount={6} labelFontSize={8} title="Resale price" />
          <YAxis labelFontSize={8} title="Remaining Lease" />
        </Section>
      </Graphic>
    </div>
    <!-- 6 -->
    <div class="graph-chart">
      <p> 6.Resale Price vs Storey Range</p>
      <Graphic width={425} height={425}>
        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleLinear().domain(sales.domain('resale_price'))}
          scaleY={scaleBand()
            .domain(salesplot.domain('storey_range'))
            .padding(0.1)} 
          >
          <PointLayer
            x={sales.column('resale_price')}
            y={sales.column('storey_range')}
            fill={color1}
            opacity={0.1} />
          <XAxis tickCount={6} labelFontSize={8} title="Resale price" />
          <YAxis labelFontSize={8} title="Storey Range" />
        </Section>
      </Graphic>
    </div>
    <!-- 7 -->
    <div class="graph-chart">
      <p> 7.Mean Resale Price vs Flat Type</p>
      <SvelteTooltip tip="Resale Price : {showinfo}" top color="#EF6C00">
        <Graphic width={425} height={425}>
          <!-- 7 -->

          <Section
            x1={0}
            x2={400}
            y1={0}
            y2={400}
            {padding}
            flipY
            scaleX={scaleBand()
                .domain(salesOverFloaType.domain('flat_type'))
                .padding(0.01)}
            }
            scaleY={scaleLinear().domain([
              0,
              salesOverFloaType.domain('mean_price')[1]
            ])}>
            <RectangleLayer
              renderSettings = {showinfo}
              onMouseover={(e)=> { ShowData(e) ;} }
              
              x1={salesOverFloaType.column('flat_type')}
              x2={({ scaleX }) => salesOverFloaType.map('flat_type', ft => scaleX(ft) + scaleX.bandwidth())}
              y1={0}
              y2={salesOverFloaType.column('mean_price')}
              
              
              fill={color1} />
            <XAxis tickCount={5} labelFontSize={8} title="Flat Type" />
            <YAxis labelFontSize={8} title="Mean Resale Price" />
          </Section>
        </Graphic>
      </SvelteTooltip>
    </div>
    <!-- 8 -->
    <div class="graph-chart">
      <p> 8.Count of Units by Flat Model</p>
      <Graphic width={425} height={425}>
        <Section
          x1={0}
          x2={400}
          y1={0}
          y2={400}
          {padding}
          flipY
          scaleX={scaleLinear().domain([
            0,
            salesflat_model.domain('total_count')[1]
          ])}
          scaleY={scaleBand()
            .domain(salesflat_model.domain('flat_model'))
            .padding(0.5)} >
          <RectangleLayer
            x1={0}
            x2={salesflat_model.column('total_count')}
            y1={salesflat_model.column('flat_model')}
            y2={({ scaleY }) => salesflat_model.map('flat_model', ft => scaleY(ft) + scaleY.bandwidth())}
            fill={color1}
            
            />
          <!-- <BarChart /> -->

          <XAxis labelFontSize={8} title="Count" />
          <YAxis labelFontSize={8} title="Flat Model" />
        </Section>

      </Graphic>
    </div>
  </div>
</div>
