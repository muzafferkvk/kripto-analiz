# kripto-analiz
import React, { useEffect, useState } from "react";
import { useRouter } from "next/router";
import { Card, CardContent } from "@/components/ui/card";
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

const fetchCryptoData = async (symbol = "BTCUSDT") => {
  const interval = "1h";
  const limit = 50;
  const url = `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=${interval}&limit=${limit}`;
  const response = await fetch(url);
  const rawData = await response.json();

  const chartData = rawData.map((item) => ({
    time: new Date(item[0]).toLocaleTimeString(),
    close: parseFloat(item[4]),
  }));

  const lastClose = chartData[chartData.length - 1].close;
  const pivot = {
    support1: lastClose * 0.98,
    support2: lastClose * 0.96,
    resistance1: lastClose * 1.02,
    resistance2: lastClose * 1.04,
  };

  return {
    pivot,
    indicators: {},
    chartData,
  };
};

const fetchIndicators = async (symbol = "BTC/USDT") => {
  const apiKey = "YOUR_TAAPI_API_KEY_HERE"; // Buraya kendi TAAPI API anahtarını ekle
  const exchange = "binance";
  const interval = "1h";

  const endpoints = {
    rsi: `https://api.taapi.io/rsi?secret=${apiKey}&exchange=${exchange}&symbol=${symbol}&interval=${interval}`,
    macd: `https://api.taapi.io/macd?secret=${apiKey}&exchange=${exchange}&symbol=${symbol}&interval=${interval}`,
    stochRsi: `https://api.taapi.io/stochrsi?secret=${apiKey}&exchange=${exchange}&symbol=${symbol}&interval=${interval}`,
  };

  const [rsiRes, macdRes, stochRsiRes] = await Promise.all([
    fetch(endpoints.rsi).then((res) => res.json()),
    fetch(endpoints.macd).then((res) => res.json()),
    fetch(endpoints.stochRsi).then((res) => res.json()),
  ]);

  return {
    rsi: rsiRes.value,
    macd: macdRes.valueMACD,
    stochRsi: stochRsiRes.valueFastK,
  };
};

export default function CoinPage() {
  const [data, setData] = useState(null);
  const router = useRouter();
  const { coin } = router.query;

  useEffect(() => {
    if (!coin) return;

    const binanceSymbol = coin.toUpperCase() + "USDT";
    const taapiSymbol = coin.toUpperCase() + "/USDT";

    const getData = async () => {
      const baseData = await fetchCryptoData(binanceSymbol);
      const indicatorData = await fetchIndicators(taapiSymbol);
      baseData.indicators = indicatorData;
      setData(baseData);
    };
    getData();
  }, [coin]);

  if (!data) return <div className="p-4">Yükleniyor...</div>;

  return (
    <div className="p-6 space-y-6 bg-gray-100 dark:bg-gray-900 min-h-screen">
      <h1 className="text-2xl font-bold text-center text-gray-800 dark:text-gray-100">
        {coin?.toUpperCase()} Teknik Analiz
      </h1>

      <Card>
        <CardContent>
          <h2 className="text-xl font-semibold mb-2">Pivot Seviyeleri</h2>
          <ul>
            <li>Destek 1: {data.pivot.support1.toFixed(2)}</li>
            <li>Destek 2: {data.pivot.support2.toFixed(2)}</li>
            <li>Direnç 1: {data.pivot.resistance1.toFixed(2)}</li>
            <li>Direnç 2: {data.pivot.resistance2.toFixed(2)}</li>
          </ul>
        </CardContent>
      </Card>

      <Card>
        <CardContent>
          <h2 className="text-xl font-semibold mb-2">Göstergeler</h2>
          <ul>
            <li>RSI: {data.indicators.rsi?.toFixed(2)}</li>
            <li>MACD: {data.indicators.macd?.toFixed(2)}</li>
            <li>StochRSI: {data.indicators.stochRsi?.toFixed(2)}</li>
          </ul>
        </CardContent>
      </Card>

      <Card>
        <CardContent>
          <h2 className="text-xl font-semibold mb-2">Fiyat Grafiği</h2>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={data.chartData}>
              <XAxis dataKey="time" />
              <YAxis />
              <Tooltip />
              <Line type="monotone" dataKey="close" stroke="#8884d8" strokeWidth={2} />
            </LineChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>
    </div>
  );
}
