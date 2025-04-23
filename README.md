# Malwarampcar
// Webowa aplikacja śledzenia przylotów na KEF
// Używa React i pobiera dane ze strony KEF przez parser backendowy (tu: mockowane dane)

import React, { useEffect, useState } from "react";
import { Button } from "@/components/ui/button";

const ROWS = 30;

export default function App() {
  const [flights, setFlights] = useState(
    Array.from({ length: ROWS }, () => ({ flightNo: "", from: "", time: "", gate: "", agent: "" }))
  );

  useEffect(() => {
    const interval = setInterval(() => {
      updateFlightInfo();
    }, 300000); // co 5 minut
    return () => clearInterval(interval);
  }, [flights]);

  const updateFlightInfo = async () => {
    const updated = await Promise.all(
      flights.map(async (flight) => {
        if (!flight.flightNo) return flight;
        try {
          const res = await fetch(`/api/fetch-flight?flight=${flight.flightNo}`);
          const data = await res.json();
          return {
            ...flight,
            from: data.from || "",
            time: data.time || "",
            gate: data.gate || "",
          };
        } catch {
          return flight;
        }
      })
    );
    setFlights(updated);
  };

  const handleInput = (index, field, value) => {
    const updated = [...flights];
    updated[index][field] = value;
    setFlights(updated);
  };

  const clearAll = () => {
    setFlights(Array.from({ length: ROWS }, () => ({ flightNo: "", from: "", time: "", gate: "", agent: "" })));
  };

  return (
    <div className="p-4 max-w-screen-xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">Przyloty KEF</h1>
      <table className="w-full table-fixed border">
        <thead>
          <tr>
            <th className="border px-2 py-1">Flight No</th>
            <th className="border px-2 py-1">From</th>
            <th className="border px-2 py-1">Time</th>
            <th className="border px-2 py-1">Gate</th>
            <th className="border px-2 py-1">Agent</th>
          </tr>
        </thead>
        <tbody>
          {flights.map((flight, i) => (
            <tr key={i}>
              <td className="border">
                <input
                  className="w-full p-1"
                  value={flight.flightNo}
                  onChange={(e) => handleInput(i, "flightNo", e.target.value)}
                />
              </td>
              <td className="border text-center">{flight.from}</td>
              <td className="border text-center">{flight.time}</td>
              <td className="border text-center">{flight.gate}</td>
              <td className="border">
                <input
                  className="w-full p-1"
                  value={flight.agent}
                  onChange={(e) => handleInput(i, "agent", e.target.value)}
                />
              </td>
            </tr>
          ))}
        </tbody>
      </table>
      <div className="mt-4">
        <Button onClick={clearAll}>Wyczyść wszystkie pola</Button>
      </div>
    </div>
  );
}

