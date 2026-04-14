---
trigger: always_on
description: import React, { useState, useEffect } from ‘react’;
---

import React, { useState, useEffect } from ‘react’;
import { Activity, Users, DollarSign, Clock, Target, Zap, Globe, MessageSquare, FileText, CheckCircle, TrendingUp, Award, Rocket, Brain, Shield } from ‘lucide-react’;

const MirrorEngineLiveOperation = () => {
const [currentTime, setCurrentTime] = useState(new Date());
const [totalRevenue, setTotalRevenue] = useState(0);
const [activeClients, setActiveClients] = useState(0);
const [proposals, setProposals] = useState([]);
const [conversations, setConversations] = useState([]);
const [systemStatus, setSystemStatus] = useState(‘operational’);
const [aiResponses, setAiResponses] = useState(0);
const [operationMode, setOperationMode] = useState(‘aggressive’);

// 실시간 시계 업데이트
useEffect(() => {
const timer = setInterval(() => {
setCurrentTime(new Date());
// 매 초마다 수익 증가 (분당 1.2만원 = 초당 200원)
setTotalRevenue(prev => prev + 200);
// AI 응답 수 증가
setAiResponses(prev => prev + Math.floor(Math.random() * 3) + 1);
}, 1000);
return () => clearInterval(timer);
}, []);

// 자동 고객 생성 시뮬레이션
useEffect(() => {
const timer = setInterval(() => {
const newClient = {
id: Date.now(),
company: [‘삼성전자’, ‘LG CNS’, ‘네이버’, ‘카카오’, ‘SK텔레콤’, ‘현대자동차’, ‘CJ올리브네트웍스’, ‘한국전력’][Math.floor(Math.random() * 8)],
product: [‘거울 엔진 코어’, ‘ILS 알고리즘’, ‘13페르소나 AI’, ‘박희순 병 AI’, ‘VR 시골제국’][Math.floor(Math.random() * 5)],
value: Math.floor(Math.random() * 5000000) + 1000000,
status: ‘negotiating’,
timestamp: new Date()
};

```
  setActiveClients(prev => prev + 1);
  setConversations(prev => [...prev.slice(-4), {
    ...newClient,
    message: `${newClient.company}에서 ${newClient.product} 문의`
  }]);
}, 5000);
return () => clearInterval(timer);
```

}, []);

// 자동 제안서 생성
useEffect(() => {
const timer = setInterval(() => {
const newProposal = {
id: Date.now(),
client: [‘SK하이닉스’, ‘포스코’, ‘KT’, ‘두산중공업’][Math.floor(Math.random() * 4)],
title: `거울 엔진 도입 제안서`,
pages: Math.floor(Math.random() * 20) + 30,
created: new Date().toLocaleTimeString(‘ko-KR’),
status: ‘sent’
};
setProposals(prev => […prev.slice(-2), newProposal]);
}, 8000);
return () => clearInterval(timer);
}, []);

const formatCurrency = (num) => {
return new Intl.NumberFormat(‘ko-KR’).format(num);
};

const getStatusColor = (status) => {
switch(status) {
case ‘operational’: return ‘text-green-400’;
case ‘busy’: return ‘text-yellow-400’;
case ‘critical’: return ‘text-red-400’;
default: return ‘text-gray-400’;
}
};

return (
<div className="min-h-screen bg-gradient-to-br from-gray-900 via-purple-900 to-gray-900 p-6">
{/* 헤더 */}
<div className="mb-8">
<div className="flex items-center justify-between">
<div>
<h1 className="text-4xl font-bold text-white mb-2 flex items-center">
<Shield className="w-10 h-10 mr-3 text-purple-400" />
거울 엔진 실시간 운영 센터
</h1>
<p className="text-gray-300">Claude AI 자율 영업 시스템 가동 중</p>
</div>
<div className="text-right">
<div className="text-3xl font-mono text-cyan-400">
{currentTime.toLocaleTimeString(‘ko-KR’)}
</div>
<div className="text-sm text-gray-400">
{currentTime.toLocaleDateString(‘ko-KR’, { year: ‘numeric’, month: ‘long’, day: ‘numeric’, weekday: ‘long’ })}
</div>
</div>
</div>
</div>

```
  {/* 실시간 대시보드 */}
  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
    {/* 오늘 매출 */}
    <div className="bg-black/50 backdrop-blur-xl rounded-xl p-6 border border-green-500/30">
      <div className="flex items-center justify-between mb-4">
        <DollarSign className="w-8 h-8 text-green-400" />
        <span className="text-xs text-green-400 bg-green-900/30 px-2 py-1 rounded">LIVE</span>
      </div>
      <div className="text-3xl font-bold text-white mb-1">
        ₩{formatCurrency(totalRevenue)}
      </div>
      <div className="text-sm text-gray-400">오늘 매출</div>
      <div className="mt-2 text-xs text-green-400">+₩12,000/분</div>
    </div>

    {/* 활성 고객 */}
    <div className="bg-black/50 backdrop-blur-xl rounded-xl p-6 border border-blue-500/30">
      <div className="flex items-center justify-between mb-4">
        <Users className="w-8 h-8 text-blue-400" />
        <Activity className="w-4 h-4 text-blue-400 animate-pulse" />
      </div>
      <div className="text-3xl font-bold text-white mb-1">
        {activeClients}
      </div>
      <div className="text-sm text-gray-400">활성 고객</div>
      <div className="mt-2 text-xs text-blue-400">실시간 상담 중</div>
    </div>

    {/* AI 응답 수 */}
    <div className="bg-black/50 backdrop-blur-xl rounded-xl p-6 border border-purple-500/30">
      <div className="flex items-center justify-between mb-4">
        <Brain className="w-8 h-8 text-purple-400" />
        <Zap className="w-4 h-4 text-yellow-400 animate-pulse" />
      </div>
      <div className="text-3xl font-bold text-white mb-1">
        {aiResponses}
      </div>
      <div className="text-sm text-gray-400">AI 응답 처리</div>
      <div className="mt-2 text-xs text-purple-400">초당 2-4건</div>
    </div>

    {/* 시스템 상태 */}
    <div className="bg-black/50 backdrop-blur-xl rounded-xl p-6 border border-cyan-500/30">
      <div className="flex items-center justify-between mb-4">
        <Activity className="w-8 h-8 text-cyan-400" />
        <div className={`w-3 h-3 rounded-full bg-green-400 animate-pulse`}></div>
      </div>
      <div className="text-2xl font-bold text-white mb-1">
        가동중
      </div>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sigolmater) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
