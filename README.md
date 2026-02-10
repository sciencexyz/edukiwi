import React, { useState, useCallback } from 'react';
import { Users, RefreshCw, Copy, Check, LayoutGrid, Move, Edit3 } from 'lucide-react';

const App = () => {
  const [startNum, setStartNum] = useState(1);
  const [endNum, setEndNum] = useState(30);
  const [peoplePerGroup, setPeoplePerGroup] = useState(4);
  const [groups, setGroups] = useState([]); // Array of arrays: [[1,2], [3,4]...]
  const [groupNames, setGroupNames] = useState([]); // Array of strings: ["모둠 1", "모둠 2"...]
  const [isCopied, setIsCopied] = useState(false);
  const [draggedItem, setDraggedItem] = useState(null); // { num, sourceGroupIndex }

  // 초기 모둠 생성
  const generateGroups = () => {
    if (startNum >= endNum) {
      alert("끝 번호는 시작 번호보다 커야 합니다.");
      return;
    }
    if (peoplePerGroup <= 0) {
      alert("모둠당 인원수는 1명 이상이어야 합니다.");
      return;
    }

    const numbers = [];
    for (let i = startNum; i <= endNum; i++) {
      numbers.push(i);
    }

    // Fisher-Yates Shuffle
    for (let i = numbers.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [numbers[i], numbers[j]] = [numbers[j], numbers[i]];
    }

    const newGroups = [];
    const newNames = [];
    for (let i = 0; i < numbers.length; i += parseInt(peoplePerGroup)) {
      newGroups.push(numbers.slice(i, i + parseInt(peoplePerGroup)));
      newNames.push(`모둠 ${newGroups.length}`);
    }

    setGroups(newGroups);
    setGroupNames(newNames);
  };

  // 모둠 이름 변경 핸들러
  const handleGroupNameChange = (index, newName) => {
    const updatedNames = [...groupNames];
    updatedNames[index] = newName;
    setGroupNames(updatedNames);
  };

  // 결과 복사 (수정된 모둠 이름 반영)
  const copyToClipboard = () => {
    if (groups.length === 0) return;
    let text = "✨ [연수 모둠 구성 결과] ✨\n\n";
    groups.forEach((group, index) => {
      if (group.length > 0) {
        const name = groupNames[index] || `모둠 ${index + 1}`;
        text += `${name}: ${group.sort((a,b) => a-b).join(', ')}\n`;
      }
    });

    const textArea = document.createElement("textarea");
    textArea.value = text;
    document.body.appendChild(textArea);
    textArea.select();
    try {
      document.execCommand('copy');
      setIsCopied(true);
      setTimeout(() => setIsCopied(false), 2000);
    } catch (err) {
      console.error('복사 실패', err);
    }
    document.body.removeChild(textArea);
  };

  // --- 드래그 앤 드롭 핸들러 ---
  const handleDragStart = (e, num, groupIndex) => {
    setDraggedItem({ num, sourceGroupIndex: groupIndex });
    e.dataTransfer.effectAllowed = "move";
  };

  const handleDragOver = (e) => {
    e.preventDefault();
    e.dataTransfer.dropEffect = "move";
  };

  const handleDrop = (e, targetGroupIndex) => {
    e.preventDefault();
    if (!draggedItem) return;

    const { num, sourceGroupIndex } = draggedItem;
    if (sourceGroupIndex === targetGroupIndex) {
      setDraggedItem(null);
      return;
    }

    const newGroups = [...groups];
    newGroups[sourceGroupIndex] = newGroups[sourceGroupIndex].filter(item => item !== num);
    newGroups[targetGroupIndex] = [...newGroups[targetGroupIndex], num];

    setGroups(newGroups);
    setDraggedItem(null);
  };

  return (
    <div className="min-h-screen bg-[#f8fafc] p-4 md:p-10 font-sans text-slate-900">
      <div className="max-w-6xl mx-auto">
        {/* 헤더 섹션 */}
        <header className="mb-10 text-center">
          <div className="inline-flex items-center justify-center w-16 h-16 bg-gradient-to-tr from-indigo-600 to-blue-500 rounded-2xl mb-4 text-white shadow-xl rotate-3 hover:rotate-0 transition-transform duration-300">
            <Users size={32} />
          </div>
          <h1 className="text-4xl font-extrabold text-slate-800 mb-3 tracking-tight">랜덤 모둠 구성기 <span className="text-blue-600">Pro</span></h1>
          <p className="text-slate-500 font-medium">모둠 이름을 클릭해서 수정하고, 번호를 드래그해서 이동시키세요.</p>
        </header>

        {/* 컨트롤 패널 */}
        <div className="bg-white rounded-[2rem] shadow-xl shadow-slate-200/60 border border-slate-100 p-8 mb-10 transition-all">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
            <div className="space-y-2">
              <label className="text-sm font-bold text-slate-600 ml-1">시작 번호</label>
              <input
                type="number"
                value={startNum}
                onChange={(e) => setStartNum(parseInt(e.target.value) || 0)}
                className="w-full px-5 py-4 rounded-2xl bg-slate-50 border-2 border-transparent focus:border-blue-500 focus:bg-white transition-all outline-none text-lg font-semibold"
              />
            </div>
            <div className="space-y-2">
              <label className="text-sm font-bold text-slate-600 ml-1">끝 번호</label>
              <input
                type="number"
                value={endNum}
                onChange={(e) => setEndNum(parseInt(e.target.value) || 0)}
                className="w-full px-5 py-4 rounded-2xl bg-slate-50 border-2 border-transparent focus:border-blue-500 focus:bg-white transition-all outline-none text-lg font-semibold"
              />
            </div>
            <div className="space-y-2">
              <label className="text-sm font-bold text-slate-600 ml-1">모둠당 인원</label>
              <input
                type="number"
                value={peoplePerGroup}
                onChange={(e) => setPeoplePerGroup(parseInt(e.target.value) || 0)}
                className="w-full px-5 py-4 rounded-2xl bg-slate-50 border-2 border-transparent focus:border-blue-500 focus:bg-white transition-all outline-none text-lg font-semibold"
              />
            </div>
          </div>

          <div className="mt-10 flex flex-col sm:flex-row gap-4">
            <button
              onClick={generateGroups}
              className="flex-[2] bg-slate-900 hover:bg-blue-600 text-white font-bold py-5 rounded-2xl flex items-center justify-center gap-3 transition-all active:scale-[0.97] shadow-lg shadow-slate-200"
            >
              <RefreshCw size={22} className="animate-spin-hover" />
              모둠 새로 짜기
            </button>
            {groups.length > 0 && (
              <button
                onClick={copyToClipboard}
                className={`flex-1 py-5 rounded-2xl font-bold flex items-center justify-center gap-3 transition-all border-2 ${
                  isCopied 
                  ? 'bg-green-50 border-green-500 text-green-600' 
                  : 'bg-white border-slate-200 text-slate-600 hover:bg-slate-50 hover:border-slate-300'
                }`}
              >
                {isCopied ? <Check size={22} /> : <Copy size={22} />}
                {isCopied ? '복사되었습니다' : '결과 복사'}
              </button>
            )}
          </div>
        </div>

        {/* 결과 섹션 */}
        {groups.length > 0 ? (
          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 animate-in fade-in slide-in-from-bottom-6 duration-700">
            {groups.map((group, groupIndex) => (
              <div 
                key={groupIndex} 
                onDragOver={handleDragOver}
                onDrop={(e) => handleDrop(e, groupIndex)}
                className={`group bg-white rounded-[1.5rem] border-2 transition-all duration-300 p-6 min-h-[200px] flex flex-col ${
                  draggedItem && draggedItem.sourceGroupIndex !== groupIndex 
                  ? 'border-dashed border-blue-300 bg-blue-50/30 scale-[1.02]' 
                  : 'border-slate-100 hover:border-blue-200 hover:shadow-xl hover:shadow-slate-200/40'
                }`}
              >
                <div className="mb-5">
                  <div className="flex items-center gap-2 group/title relative">
                    <span className="w-1.5 h-5 bg-blue-500 rounded-full"></span>
                    <input
                      type="text"
                      value={groupNames[groupIndex] || ''}
                      onChange={(e) => handleGroupNameChange(groupIndex, e.target.value)}
                      className="bg-transparent border-none focus:ring-0 font-black text-slate-700 text-lg tracking-tight w-full p-0 outline-none hover:bg-slate-50 rounded px-1 transition-colors"
                      placeholder="모둠 이름 입력"
                    />
                    <Edit3 size={14} className="text-slate-300 opacity-0 group-hover/title:opacity-100 transition-opacity absolute right-0" />
                  </div>
                  <div className="mt-1 flex justify-between items-center">
                    <span className="text-slate-400 text-[10px] font-bold uppercase tracking-widest">
                      {group.length} Persons
                    </span>
                  </div>
                </div>
                
                <div className="flex flex-wrap gap-3 grow content-start">
                  {group.length > 0 ? (
                    group.map((num) => (
                      <div 
                        key={num} 
                        draggable
                        onDragStart={(e) => handleDragStart(e, num, groupIndex)}
                        className="w-12 h-12 flex items-center justify-center bg-white text-slate-800 font-bold rounded-2xl text-base shadow-sm border border-slate-100 cursor-grab active:cursor-grabbing hover:bg-indigo-600 hover:text-white hover:scale-110 hover:-translate-y-1 transition-all group-hover:shadow-md ring-4 ring-transparent hover:ring-indigo-100"
                      >
                        {num}
                      </div>
                    ))
                  ) : (
                    <div className="w-full h-24 flex items-center justify-center text-slate-300 italic text-sm border-2 border-dashed border-slate-50 rounded-xl bg-slate-50/30">
                      비어있음
                    </div>
                  )}
                </div>
              </div>
            ))}
          </div>
        ) : (
          <div className="text-center py-32 bg-white/50 rounded-[3rem] border-4 border-dashed border-slate-200">
            <div className="bg-slate-100 w-20 h-20 rounded-full flex items-center justify-center mx-auto mb-6">
              <LayoutGrid className="text-slate-300" size={40} />
            </div>
            <p className="text-slate-400 font-medium text-lg">상단 설정을 마친 후 모둠을 생성해 주세요.</p>
          </div>
        )}

        {/* 안내 */}
        {groups.length > 0 && (
          <div className="mt-12 flex flex-col items-center gap-3 text-slate-400 text-sm font-medium">
            <div className="flex items-center gap-2">
              <Move size={16} />
              <span>번호를 드래그하여 다른 모둠으로 옮길 수 있습니다.</span>
            </div>
            <div className="flex items-center gap-2">
              <Edit3 size={16} />
              <span>모둠 이름을 클릭하여 직접 수정할 수 있습니다.</span>
            </div>
          </div>
        )}

        <footer className="mt-20 text-center border-t border-slate-200 pt-10 pb-10">
          <p className="text-slate-400 text-sm font-medium">
            © 2024 교직원 연수 도구 • <span className="text-slate-300">Designed for Educational Excellence</span>
          </p>
        </footer>
      </div>

      <style>{`
        .animate-spin-hover:hover {
          animation: spin 1s ease-in-out infinite;
        }
        @keyframes spin {
          from { transform: rotate(0deg); }
          to { transform: rotate(360deg); }
        }
        input::-webkit-outer-spin-button,
        input::-webkit-inner-spin-button {
          -webkit-appearance: none;
          margin: 0;
        }
      `}</style>
    </div>
  );
};

export default App;
