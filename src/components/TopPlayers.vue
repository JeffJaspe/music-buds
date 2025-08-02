import React, { useEffect, useState } from "react";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Card, CardContent } from "@/components/ui/card";
import axios from "axios";
import { Badge } from "@/components/ui/badge";

const divisions = ["americas", "europe", "se_asia", "china"];
const rankMap = {
  10: "Herald",
  20: "Guardian",
  30: "Crusader",
  40: "Archon",
  50: "Legend",
  60: "Ancient",
  70: "Divine",
  80: "Immortal",
};

const rankColors = {
  Herald: "bg-gray-500",
  Guardian: "bg-green-500",
  Crusader: "bg-blue-500",
  Archon: "bg-indigo-500",
  Legend: "bg-yellow-500",
  Ancient: "bg-orange-500",
  Divine: "bg-red-500",
  Immortal: "bg-purple-700",
};

export default function DotaLeaderboard() {
  const [data, setData] = useState({});

  useEffect(() => {
    divisions.forEach(async (division) => {
      try {
        const res = await axios.get(
          `https://api.opendota.com/api/leaderboards?division=${division}`
        );
        setData((prev) => ({ ...prev, [division]: res.data.leaderboard || [] }));
      } catch (err) {
        console.error(`Error fetching ${division}`, err);
      }
    });
  }, []);

  const groupByRank = (players) => {
    const grouped = {};
    players.forEach((player) => {
      const tier = Math.floor((player.rank_tier || 0) / 10) * 10;
      const label = rankMap[tier] || "Unranked";
      if (!grouped[label]) grouped[label] = [];
      grouped[label].push(player);
    });
    return grouped;
  };

  return (
    <Tabs defaultValue="americas" className="w-full">
      <TabsList className="flex flex-wrap justify-center gap-2 p-4">
        {divisions.map((div) => (
          <TabsTrigger
            key={div}
            value={div}
            className="text-xl px-6 py-3 rounded-2xl shadow-lg bg-slate-800 text-white"
          >
            {div.toUpperCase()}
          </TabsTrigger>
        ))}
      </TabsList>

      {divisions.map((div) => (
        <TabsContent key={div} value={div}>
          <div className="p-4">
            {Object.entries(groupByRank(data[div] || [])).map(([rank, players]) => (
              <div key={rank} className="mb-8">
                <h2 className={`text-2xl font-bold mb-2 ${rankColors[rank]}`}>{rank}</h2>
                <div className="overflow-x-auto">
                  <table className="min-w-full bg-white rounded shadow-md">
                    <thead className="bg-slate-200">
                      <tr>
                        <th className="px-4 py-2 text-left">Rank</th>
                        <th className="px-4 py-2 text-left">Name</th>
                        <th className="px-4 py-2 text-left">MMR</th>
                      </tr>
                    </thead>
                    <tbody>
                      {players.map((p, idx) => (
                        <tr key={p.account_id} className="border-b hover:bg-slate-100">
                          <td className="px-4 py-2">#{idx + 1}</td>
                          <td className="px-4 py-2">{p.name || "Anonymous"}</td>
                          <td className="px-4 py-2">{p.mmr}</td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            ))}
          </div>
        </TabsContent>
      ))}
    </Tabs>
  );
}
