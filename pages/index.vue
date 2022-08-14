<template>
    <div>
        <header class="flex justify-center">
            <h1>코로나 세계 현황판</h1>
        </header>
        <main class="flex">
            <div class="left-panel flex column">
                <div class="total-board">
                    <p>Total Confirmed</p>
                    <span class="confirmed-total"></span>
                </div>
                <div class="country-ranks">
                    <p>Confirmed Cases by Country</p>
                    <ol class="rank-list"></ol>
                </div>
                <p
                    class="last-updated-time flex justify-center align-center"
                ></p>
            </div>
            <div class="right-panel">
                <div class="summary-wrapper flex">
                    <div class="deaths-container">
                        <h3 class="summary-title">Total Deaths</h3>
                        <p class="total deaths">0</p>
                        <div class="list-wrapper">
                            <ol class="deaths-list"></ol>
                        </div>
                    </div>
                    <div class="recovered-container">
                        <h3 class="summary-title">Total Recovered</h3>
                        <p class="total recovered">0</p>
                        <div class="list-wrapper">
                            <ol class="recovered-list"></ol>
                        </div>
                    </div>
                </div>
                <div class="chart-container">
                    <canvas
                        id="lineChart"
                        class="corona-chart"
                        style="
                            width: 100%;
                            height: 356px;
                            background-color: #5b5656;
                        "
                    ></canvas>
                </div>
            </div>
        </main>
    </div>
</template>

<script setup lang="ts">
import { onMounted } from "vue";
import axios from "axios";
import { Chart } from "chart.js";

onMounted(() => {
    // utils
    function $(selector: string) {
        return document.querySelector(selector);
    }
    function getUnixTimestamp(date: Date) {
        return new Date(date).getTime();0
    }

    // DOM                  // $ = document.querySelector
    const confirmedTotal = $(".confirmed-total") as HTMLSpanElement;
    const deathsTotal = $(".deaths") as HTMLParagraphElement;
    const recoveredTotal = $(".recovered") as HTMLParagraphElement;
    const lastUpdatedTime = $(".last-updated-time") as HTMLParagraphElement;
    const rankList = $(".rank-list");
    const deathsList = $(".deaths-list");
    const recoveredList = $(".recovered-list");
    const deathSpinner = createSpinnerElement("deaths-spinner");
    const recoveredSpinner = createSpinnerElement("recovered-spinner");

    function createSpinnerElement(id: string) {
        const wrapperDiv = document.createElement("div");
        wrapperDiv.setAttribute("id", id);
        wrapperDiv.setAttribute(
            "class",
            "spinner-wrapper flex justify-center align-center"
        );
        const spinnerDiv = document.createElement("div");
        spinnerDiv.setAttribute("class", "ripple-spinner");
        spinnerDiv.appendChild(document.createElement("div"));
        spinnerDiv.appendChild(document.createElement("div"));
        wrapperDiv.appendChild(spinnerDiv);
        return wrapperDiv;
    }

    // state
    let isDeathLoading = false;
    // eslint-disable-next-line no-unused-vars
    let isRecoveredLoading = false;

    function fetchCovidSummary() {
        const url = "https://api.covid19api.com/summary";
        return axios.get(url);
    }

    enum CovidStatus {
        Confirmed = 'confirmed',
        Recovered = 'recovered',
        Deaths = 'deaths'
    }

    function fetchCountryInfo(countryCode: string, status: CovidStatus) {
        // params: confirmed, recovered, deaths
        const url = `https://api.covid19api.com/country/${countryCode}/status/${status}`;
        return axios.get(url);
    }

    // methods
    function startApp() {
        setupData();
        initEvents();
    }

    // events
    function initEvents() {
        rankList.addEventListener("click", handleListClick);
    }

    async function handleListClick(event: any) {
        let selectedId;
        if (
            event.target instanceof HTMLParagraphElement ||
            event.target instanceof HTMLSpanElement
        ) {
            selectedId = event.target.parentElement.id;
        }
        if (event.target instanceof HTMLLIElement) {
            selectedId = event.target.id;
        }
        if (isDeathLoading) {
            return;
        }
        clearDeathList();
        clearRecoveredList();
        startLoadingAnimation();
        isDeathLoading = true;
        const { data: deathResponse } = await fetchCountryInfo(
            selectedId, CovidStatus.Deaths
            );
        const { data: recoveredResponse } = await fetchCountryInfo(
            selectedId, CovidStatus.Recovered
        );
        const { data: confirmedResponse } = await fetchCountryInfo(
            selectedId, CovidStatus.Confirmed
        );
        endLoadingAnimation();
        setDeathsList(deathResponse);
        setTotalDeathsByCountry(deathResponse);
        setRecoveredList(recoveredResponse);
        setTotalRecoveredByCountry(recoveredResponse);
        setChartData(confirmedResponse);
        isDeathLoading = false;
    }

    function setDeathsList(data: any) {
        const sorted = data.sort(
            (a: any, b: any) => getUnixTimestamp(b.Date) - getUnixTimestamp(a.Date)
        );
        sorted.forEach((value: any) => {
            const li = document.createElement("li");
            li.setAttribute("class", "list-item-b flex align-center");
            const span = document.createElement("span");
            span.textContent = value.Cases;
            span.setAttribute("class", "deaths");
            const p = document.createElement("p");
            p.textContent = new Date(value.Date)
                .toLocaleDateString()
                .slice(0, -1);
            li.appendChild(span);
            li.appendChild(p);
            deathsList.appendChild(li);
        });
    }

    function clearDeathList() {
        deathsList.innerHTML = null;
    }

    function setTotalDeathsByCountry(data: any) {
        deathsTotal.innerText = data[0].Cases;
    }

    function setRecoveredList(data: any) {
        const sorted = data.sort(
            (a: any, b: any) => getUnixTimestamp(b.Date) - getUnixTimestamp(a.Date)
        );
        sorted.forEach((value: any) => {
            const li = document.createElement("li");
            li.setAttribute("class", "list-item-b flex align-center");
            const span = document.createElement("span");
            span.textContent = value.Cases;
            span.setAttribute("class", "recovered");
            const p = document.createElement("p");
            p.textContent = new Date(value.Date)
                .toLocaleDateString()
                .slice(0, -1);
            li.appendChild(span);
            li.appendChild(p);
            recoveredList.appendChild(li);
        });
    }

    function clearRecoveredList() {
        recoveredList.innerHTML = null;
    }

    function setTotalRecoveredByCountry(data: any) {
        recoveredTotal.innerText = data[0].Cases;
    }

    function startLoadingAnimation() {
        deathsList.appendChild(deathSpinner);
        recoveredList.appendChild(recoveredSpinner);
    }

    function endLoadingAnimation() {
        deathsList.removeChild(deathSpinner);
        recoveredList.removeChild(recoveredSpinner);
    }

    async function setupData() {
        const { data } = await fetchCovidSummary();
        setTotalConfirmedNumber(data);
        setTotalDeathsByWorld(data);
        setTotalRecoveredByWorld(data);
        setCountryRanksByConfirmedCases(data);
        setLastUpdatedTimestamp(data);
    }

    function renderChart(data: any, labels: any) {
        const ctx = $("#lineChart").getContext("2d");
        Chart.defaults.color = "#f5eaea";
        Chart.defaults.font.family = "Exo 2";
        new Chart(ctx, {
            type: "line",
            data: {
                labels,
                datasets: [
                    {
                        label: "Confirmed for the last two weeks",
                        backgroundColor: "#feb72b",
                        borderColor: "#feb72b",
                        data,
                    },
                ],
            },
            options: {},
        });
    }

    function setChartData(data: any) {
        const chartData = data.slice(-14).map((value: any) => value.Cases);
        const chartLabel = data
            .slice(-14)
            .map((value: any) =>
                new Date(value.Date).toLocaleDateString().slice(5, -1)
            );
        renderChart(chartData, chartLabel);
    }

    function setTotalConfirmedNumber(data: any) {
        confirmedTotal.innerText = data.Countries.reduce(
            (total: any, current: any) => (total += current.TotalConfirmed),
            0
        );
    }

    function setTotalDeathsByWorld(data: any) {
        deathsTotal.innerText = data.Countries.reduce(
            (total: any, current: any) => (total += current.TotalDeaths),
            0
        );
    }

    function setTotalRecoveredByWorld(data: any) {
        recoveredTotal.innerText = data.Countries.reduce(
            (total: any, current: any) => (total += current.TotalRecovered),
            0
        );
    }

    function setCountryRanksByConfirmedCases(data: any) {
        const sorted = data.Countries.sort(
            (a: any, b: any) => b.TotalConfirmed - a.TotalConfirmed
        );
        sorted.forEach((value: any) => {
            const li = document.createElement("li");
            li.setAttribute("class", "list-item flex align-center");
            li.setAttribute("id", value.Slug);
            const span = document.createElement("span");
            span.textContent = value.TotalConfirmed;
            span.setAttribute("class", "cases");
            const p = document.createElement("p");
            p.setAttribute("class", "country");
            p.textContent = value.Country;
            li.appendChild(span);
            li.appendChild(p);
            rankList.appendChild(li);
        });
    }

    function setLastUpdatedTimestamp(data: any) {
        lastUpdatedTime.innerText = new Date(data.Date).toLocaleString();
    }

    startApp();
});
</script>

<style lang="scss"></style>
