import React, { useEffect, useRef, useState } from 'react';
import { ChevronDown, ChevronUp, Zap, BarChart2, Award, ArrowRight, Layout, Cpu, Target, Image as ImageIcon, Upload, CheckCircle, Plus, PenTool, Layers, Monitor, X, Trash2, RefreshCw, ChevronLeft, ChevronRight } from 'lucide-react';

// --- 视觉配置常量 ---
const VISUAL_CONFIG = {
  colors: {
    bg: '#050A18',
    primary: '#0247F0',
    secondary: '#784BC5',
    textMain: '#FFFFFF',
    textMuted: 'rgba(255, 255, 255, 0.6)',
  },
  particle: {
    count: 1500,
    baseSize: 1.2,
    friction: 0.92,
    ease: 0.08,
    mouseForce: 100,
    mouseRadius: 150,
    pulseForce: 300,
  },
  torus: {
    radiusMain: 180,
    radiusTube: 70,
  }
};

// --- 辅助组件：打字机效果 ---
const TypewriterText = ({ text, delay = 0, speed = 80, className = "" }) => {
  const [displayText, setDisplayText] = useState('');
  const [showCursor, setShowCursor] = useState(true);

  useEffect(() => {
    let timeout;
    let currentIndex = 0;
    const startTimeout = setTimeout(() => {
      const typeChar = () => {
        if (currentIndex < text.length) {
          setDisplayText(text.slice(0, currentIndex + 1));
          currentIndex++;
          timeout = setTimeout(typeChar, speed);
        }
      };
      typeChar();
    }, delay);
    const cursorInterval = setInterval(() => setShowCursor(p => !p), 500);
    return () => {
      clearTimeout(startTimeout);
      clearTimeout(timeout);
      clearInterval(cursorInterval);
    };
  }, [text, delay, speed]);

  return (
    <span className={className}>
      {displayText}
      <span className={`${showCursor ? 'opacity-100' : 'opacity-0'} ml-1 text-purple-400`}>|</span>
    </span>
  );
};

// --- 辅助组件：支持多图轮播、删除、替换、左右切换及拖拽上传的上传组件 ---
const TechImagePlaceholder = ({ label, subLabel, height = "aspect-video", maxImages = 8, onZoom }) => {
  const [images, setImages] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [isDragging, setIsDragging] = useState(false); // 拖拽状态
  const fileInputRef = useRef(null);      // 用於新增圖片
  const replaceInputRef = useRef(null);   // 用於替換單張圖片

  // 自動輪播邏輯
  useEffect(() => {
    if (images.length <= 1 || isDragging) return; // 拖拽时不轮播
    const interval = setInterval(() => {
      setCurrentIndex((prev) => (prev + 1) % images.length);
    }, 3000);
    return () => clearInterval(interval);
  }, [images.length, currentIndex, isDragging]);

  // 处理文件列表
  const processFiles = (files) => {
    if (!files || files.length === 0) return;

    Promise.all(Array.from(files).map(file => {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target.result);
        reader.readAsDataURL(file);
      });
    })).then(newImages => {
      setImages(prev => {
        const combined = [...prev, ...newImages];
        return combined.slice(0, maxImages);
      });
      // 如果之前沒有圖片，重置索引
      if (images.length === 0) setCurrentIndex(0);
    });
  };

  // 新增圖片点击
  const handleAddClick = (e) => {
    e.stopPropagation();
    fileInputRef.current.click();
  };

  // Input Change 事件
  const handleFileChange = (e) => {
    processFiles(e.target.files);
    e.target.value = ''; // 重置 input
  };

  // --- 拖拽事件处理 ---
  const handleDragEnter = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(true);
  };

  const handleDragOver = (e) => {
    e.preventDefault();
    e.stopPropagation();
    if (!isDragging) setIsDragging(true);
  };

  const handleDragLeave = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
  };

  const handleDrop = (e) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
    
    if (e.dataTransfer.files && e.dataTransfer.files.length > 0) {
      processFiles(e.dataTransfer.files);
      e.dataTransfer.clearData();
    }
  };

  // 刪除當前圖片
  const handleDeleteCurrent = (e) => {
    e.stopPropagation();
    if (images.length === 0) return;

    setImages(prev => {
      const newImages = prev.filter((_, i) => i !== currentIndex);
      if (currentIndex >= newImages.length) {
        setCurrentIndex(Math.max(0, newImages.length - 1));
      }
      return newImages;
    });
  };

  // 觸發替換當前圖片
  const handleReplaceClick = (e) => {
    e.stopPropagation();
    replaceInputRef.current.click();
  };

  // 替換圖片處理
  const handleReplaceFileChange = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (ev) => {
      setImages(prev => {
        const newImages = [...prev];
        newImages[currentIndex] = ev.target.result;
        return newImages;
      });
    };
    reader.readAsDataURL(file);
    e.target.value = ''; // 重置 input
  };

  // 上一張圖片
  const handlePrevClick = (e) => {
    e.stopPropagation();
    setCurrentIndex((prev) => (prev - 1 + images.length) % images.length);
  };

  // 下一張圖片
  const handleNextClick = (e) => {
    e.stopPropagation();
    setCurrentIndex((prev) => (prev + 1) % images.length);
  };

  // 圖片點擊 (放大)
  const handleImageClick = (e, imgSrc) => {
    e.stopPropagation();
    if (onZoom) {
      onZoom(imgSrc, label);
    }
  };

  return (
    <div
      className={`relative w-full ${height} bg-[#0A1124] rounded-lg overflow-hidden border transition-all duration-300 active:scale-[0.98] select-none
        ${isDragging 
          ? 'border-blue-500 border-dashed bg-blue-900/20 shadow-[0_0_30px_rgba(59,130,246,0.3)] scale-[1.02]' 
          : 'border-white/10 hover:border-blue-500/50 hover:shadow-lg hover:shadow-blue-900/20'
        } group`}
      onClick={(e) => {
        // 如果沒有圖片，點擊整個區域觸發上傳
        if (images.length === 0) handleAddClick(e);
      }}
      onDragEnter={handleDragEnter}
      onDragOver={handleDragOver}
      onDragLeave={handleDragLeave}
      onDrop={handleDrop}
    >
      {/* 隱藏的文件輸入框 */}
      <input
        type="file"
        multiple
        ref={fileInputRef}
        onChange={handleFileChange}
        accept="image/*"
        className="hidden"
      />
      <input
        type="file"
        ref={replaceInputRef}
        onChange={handleReplaceFileChange}
        accept="image/*"
        className="hidden"
      />

      {/* 拖拽时的覆盖层提示 */}
      {isDragging && (
        <div className="absolute inset-0 z-50 flex flex-col items-center justify-center bg-black/60 backdrop-blur-sm pointer-events-none animate-pulse">
          <Upload size={48} className="text-blue-400 mb-4" />
          <p className="text-blue-200 text-lg font-bold">释放以上传图片</p>
        </div>
      )}

      {images.length > 0 ? (
        <>
           {/* 圖片展示 */}
           {images.map((img, index) => (
             <img 
               key={index}
               src={img} 
               alt={`${label}-${index}`} 
               className={`absolute inset-0 w-full h-full object-cover transition-opacity duration-700 cursor-zoom-in ${index === currentIndex ? 'opacity-80' : 'opacity-0'}`} 
               onClick={(e) => handleImageClick(e, img)}
             />
           ))}
           
           {/* 遮罩漸變 */}
           <div className="absolute inset-0 bg-gradient-to-t from-[#050A18] via-transparent to-transparent opacity-60 pointer-events-none"></div>
           
           {/* 右上角計數器 */}
           <div className="absolute top-2 right-3 bg-black/50 backdrop-blur-sm px-2 py-0.5 rounded text-[10px] text-white/80 font-mono border border-white/10 pointer-events-none z-10">
             {currentIndex + 1} / {images.length}
           </div>

           {/* 頂部操作欄 (刪除與替換) */}
           <div className="absolute top-2 left-3 flex gap-2 z-20 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
              <button 
                onClick={handleReplaceClick}
                className="p-1.5 bg-black/50 hover:bg-blue-600 rounded-md text-white/80 hover:text-white transition-colors border border-white/10"
                title="替換當前圖片"
              >
                <RefreshCw size={14} />
              </button>
              <button 
                onClick={handleDeleteCurrent}
                className="p-1.5 bg-black/50 hover:bg-red-600 rounded-md text-white/80 hover:text-white transition-colors border border-white/10"
                title="刪除當前圖片"
              >
                <Trash2 size={14} />
              </button>
           </div>

           {/* 左右切換箭頭 */}
           {images.length > 1 && (
             <>
               <button
                 onClick={handlePrevClick}
                 className="absolute left-2 top-1/2 -translate-y-1/2 p-2 bg-black/20 hover:bg-black/60 backdrop-blur-sm border border-white/5 hover:border-white/20 rounded-full text-white/70 hover:text-white transition-all opacity-0 group-hover:opacity-100 z-40 transform hover:scale-110"
                 title="上一張"
               >
                 <ChevronLeft size={18} />
               </button>
               <button
                 onClick={handleNextClick}
                 className="absolute right-2 top-1/2 -translate-y-1/2 p-2 bg-black/20 hover:bg-black/60 backdrop-blur-sm border border-white/5 hover:border-white/20 rounded-full text-white/70 hover:text-white transition-all opacity-0 group-hover:opacity-100 z-40 transform hover:scale-110"
                 title="下一張"
               >
                 <ChevronRight size={18} />
               </button>
             </>
           )}

           {/* 底部指示點 */}
           <div className="absolute bottom-2 left-0 right-0 flex justify-center gap-1.5 z-10 px-4 overflow-hidden pointer-events-none">
             {images.map((_, idx) => (
               <div key={idx} className={`h-1 rounded-full transition-all shrink-0 ${idx === currentIndex ? 'bg-white w-3' : 'bg-white/30 w-1'}`} />
             ))}
           </div>
        </>
      ) : (
        // 空狀態顯示
        <div className="absolute inset-0 bg-gradient-to-br from-white/5 to-white/0 flex flex-col items-center justify-center text-white/30 group-hover:text-white/60 transition-colors cursor-pointer">
          <ImageIcon size={28} className="mb-2" />
          <span className="text-xs tracking-widest uppercase">{label}</span>
          <span className="text-[10px] text-white/20 mt-2 bg-white/5 px-2 py-0.5 rounded-full border border-white/5">點擊或拖曳上傳 (最多{maxImages}張)</span>
        </div>
      )}

      {/* 中心懸停層與添加按鈕 */}
      <div className={`absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity duration-300 backdrop-blur-[1px] z-10 pointer-events-none ${images.length === 0 ? 'hidden' : ''}`}></div>
      
      <div className={`absolute inset-0 flex flex-col items-center justify-center opacity-0 group-hover:opacity-100 transition-all duration-300 transform translate-y-4 group-hover:translate-y-0 z-30 pointer-events-none ${images.length === 0 ? 'hidden' : ''}`}>
        <div 
          onClick={handleAddClick}
          className="bg-blue-600 p-3 rounded-full mb-2 shadow-lg hover:bg-blue-500 transition-colors flex items-center justify-center cursor-pointer pointer-events-auto"
          title="新增圖片"
        >
           <Plus size={20} className="text-white" />
        </div>
        <p className="text-white font-bold tracking-wide text-sm">新增圖片</p>
        <p className="text-[10px] text-gray-300 mt-1">支持拖曳上傳 · 點擊背景放大</p>
      </div>
    </div>
  );
};

// --- 辅助组件：可展开的列表项 ---
const ExpandableItem = ({ id, title, subtitle, description, activeId, onToggle, onZoom, children }) => {
  const isOpen = activeId === id;

  return (
    <div 
      className={`p-4 rounded-lg border transition-all duration-300 cursor-pointer ${
        isOpen 
          ? 'bg-white/10 border-purple-500/50 shadow-lg shadow-purple-900/10' 
          : 'bg-white/5 border-white/5 hover:bg-white/10 hover:border-purple-500/30'
      }`}
      onClick={() => onToggle(id)}
    >
      <div className="flex justify-between items-center mb-1">
        <strong className="block text-white text-lg">{title}</strong>
        <div className={`transition-transform duration-300 ${isOpen ? 'rotate-180 text-purple-400' : 'text-gray-500'}`}>
           <ChevronDown size={18} />
        </div>
      </div>
      <p className="text-gray-300 text-sm mb-2">{subtitle}</p>
      <p className="text-gray-400 text-xs leading-relaxed opacity-80">{description}</p>
      
      {/* 展开内容区域 */}
      <div 
        className={`grid transition-all duration-500 ease-in-out ${
          isOpen ? 'grid-rows-[1fr] opacity-100 mt-4' : 'grid-rows-[0fr] opacity-0'
        }`}
      >
        <div className="overflow-hidden cursor-auto" onClick={(e) => e.stopPropagation()}>
          <div className="pt-4 border-t border-white/10">
            {children && (
              <div className="mb-4 space-y-3 text-xs text-gray-300 bg-black/20 p-3 rounded-lg border border-white/5">
                {children}
              </div>
            )}

            <div className="flex items-center gap-2 mb-3 mt-4">
              <span className="w-1 h-4 bg-purple-500 rounded-full"></span>
              <span className="text-xs font-bold text-purple-300 uppercase tracking-wider">Project Gallery</span>
            </div>
            <TechImagePlaceholder label={`${title} - 成果展示`} subLabel="点击上传相关成果图" onZoom={onZoom} />
          </div>
        </div>
      </div>
    </div>
  );
};

// --- 辅助组件：章节标题 ---
const SectionTitle = ({ number, title, icon: Icon }) => (
  <div className="flex items-center gap-4 mb-10 border-b border-white/10 pb-4">
    <div className="flex items-center justify-center w-12 h-12 rounded-xl bg-gradient-to-br from-[#0247F0] to-[#784BC5] shadow-lg shadow-purple-900/20">
      <Icon className="text-white" size={24} />
    </div>
    <div>
      <h2 className="text-3xl font-bold text-white tracking-tight">
        <span className="text-purple-400 mr-2">0{number}.</span>
        {title}
      </h2>
    </div>
  </div>
);

// --- 主应用组件 ---
export default function App() {
  const canvasRef = useRef(null);
  const [scrollY, setScrollY] = useState(0);
  const [activeSection, setActiveSection] = useState('home');
  const [expandedId, setExpandedId] = useState(null);
  
  // Modal 状态
  const [modalData, setModalData] = useState({ isOpen: false, src: '', alt: '' });

  const toggleExpand = (id) => {
    setExpandedId(prev => prev === id ? null : id);
  };

  const openModal = (src, alt) => {
    setModalData({ isOpen: true, src, alt });
  };

  const closeModal = () => {
    setModalData({ ...modalData, isOpen: false });
  };

  useEffect(() => {
    const handleScroll = () => {
      setScrollY(window.scrollY);
      const sections = ['summary', 'ai-experience', 'future-plan'];
      for (const section of sections) {
        const el = document.getElementById(section);
        if (el && window.scrollY >= (el.offsetTop - 200)) {
          setActiveSection(section);
        } else if (window.scrollY < 500) {
            setActiveSection('home');
        }
      }
    };
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  const scrollToSection = (id) => {
    const el = document.getElementById(id);
    if (el) {
      window.scrollTo({
        top: el.offsetTop - 100,
        behavior: 'smooth'
      });
    }
  };

  // --- Canvas 3D 逻辑 ---
  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    let animationFrameId;
    let width, height;
    
    const mouse = { x: -1000, y: -1000, active: false };
    let clickPulse = 0;
    let particles = [];
    let angleX = 0, angleY = 0;

    const resize = () => {
      width = window.innerWidth;
      height = window.innerHeight;
      canvas.width = width;
      canvas.height = height;
    };
    window.addEventListener('resize', resize);
    resize();

    const handleMouseMove = (e) => {
      const rect = canvas.getBoundingClientRect();
      mouse.x = e.clientX - rect.left;
      mouse.y = e.clientY - rect.top;
      mouse.active = true;
    };
    
    const handleTouchMove = (e) => {
        if(e.touches.length > 0) {
            const rect = canvas.getBoundingClientRect();
            mouse.x = e.touches[0].clientX - rect.left;
            mouse.y = e.touches[0].clientY - rect.top;
            mouse.active = true;
        }
    }

    const handleClick = () => { clickPulse = VISUAL_CONFIG.particle.pulseForce; }
    const handleLeave = () => { mouse.active = false; mouse.x = -1000; mouse.y = -1000; };

    window.addEventListener('mousemove', handleMouseMove);
    window.addEventListener('touchmove', handleTouchMove, { passive: false });
    window.addEventListener('mousedown', handleClick);
    window.addEventListener('touchstart', handleClick);
    window.addEventListener('mouseout', handleLeave);
    window.addEventListener('touchend', handleLeave);

    class Particle {
      constructor(theta, phi) {
        this.theta = theta; this.phi = phi;
        this.updateBasePosition();
        this.x = this.baseX; this.y = this.baseY; this.z = this.baseZ;
        this.vx = 0; this.vy = 0;
        this.colorBase = Math.random() > 0.4 ? VISUAL_CONFIG.colors.primary : VISUAL_CONFIG.colors.secondary;
        this.size = Math.random() * VISUAL_CONFIG.particle.baseSize + 0.5;
      }
      updateBasePosition() {
        const R = VISUAL_CONFIG.torus.radiusMain;
        const r = VISUAL_CONFIG.torus.radiusTube;
        this.baseX = (R + r * Math.cos(this.phi)) * Math.cos(this.theta);
        this.baseY = (R + r * Math.cos(this.phi)) * Math.sin(this.theta);
        this.baseZ = r * Math.sin(this.phi);
      }
      update(rotationX, rotationY) {
        let x1 = this.baseX * Math.cos(rotationY) - this.baseZ * Math.sin(rotationY);
        let z1 = this.baseX * Math.sin(rotationY) + this.baseZ * Math.cos(rotationY);
        let y1 = this.baseY * Math.cos(rotationX) - z1 * Math.sin(rotationX);
        let z2 = this.baseY * Math.sin(rotationX) + z1 * Math.cos(rotationX);
        const fov = 400; const scale = fov / (fov + z2);
        const centerX = width / 2; const centerY = height / 2;
        const screenX = centerX + x1 * scale; const screenY = centerY + y1 * scale;
        let dx = mouse.x - screenX; let dy = mouse.y - screenY;
        let distance = Math.sqrt(dx * dx + dy * dy);
        const maxDistance = VISUAL_CONFIG.particle.mouseRadius;
        if (distance < maxDistance && mouse.active) {
            let force = (maxDistance - distance) / maxDistance;
            force = -force * VISUAL_CONFIG.particle.mouseForce;
            this.vx += (dx / distance) * force;
            this.vy += (dy / distance) * force;
        }
        if (clickPulse > 1) {
            let dxC = screenX - centerX; let dyC = screenY - centerY;
            let distC = Math.sqrt(dxC*dxC + dyC*dyC);
            if (distC > 0) {
                 this.vx += (dxC / distC) * clickPulse * (Math.random() * 0.5 + 0.5);
                 this.vy += (dyC / distC) * clickPulse * (Math.random() * 0.5 + 0.5);
            }
        }
        this.vx *= VISUAL_CONFIG.particle.friction; this.vy *= VISUAL_CONFIG.particle.friction;
        if (!this.offsetX) this.offsetX = 0; if (!this.offsetY) this.offsetY = 0;
        this.offsetX += this.vx; this.offsetY += this.vy;
        const springForceX = -this.offsetX * VISUAL_CONFIG.particle.ease;
        const springForceY = -this.offsetY * VISUAL_CONFIG.particle.ease;
        this.vx += springForceX; this.vy += springForceY;
        const finalX = screenX + this.offsetX; const finalY = screenY + this.offsetY;
        const finalSize = this.size * scale;
        const alpha = Math.max(0.1, (scale - 0.5) * 1.5); 
        ctx.beginPath(); ctx.arc(finalX, finalY, Math.max(0, finalSize), 0, Math.PI * 2);
        ctx.fillStyle = this.colorBase; ctx.globalAlpha = alpha; ctx.fill(); ctx.globalAlpha = 1;
      }
    }

    const initParticles = () => {
      particles = [];
      for (let i = 0; i < VISUAL_CONFIG.particle.count; i++) {
        particles.push(new Particle(Math.random() * Math.PI * 2, Math.random() * Math.PI * 2));
      }
    };
    initParticles();

    const animate = () => {
      ctx.clearRect(0, 0, width, height); angleX += 0.002; angleY += 0.003;
      if (clickPulse > 0) clickPulse *= 0.9;
      particles.forEach(p => p.update(angleX, angleY));
      animationFrameId = requestAnimationFrame(animate);
    };
    animate();

    return () => {
      window.removeEventListener('resize', resize);
      window.removeEventListener('mousemove', handleMouseMove);
      window.removeEventListener('touchmove', handleTouchMove);
      window.removeEventListener('mousedown', handleClick);
      window.removeEventListener('touchstart', handleClick);
      window.removeEventListener('mouseout', handleLeave);
      window.removeEventListener('touchend', handleLeave);
      cancelAnimationFrame(animationFrameId);
    };
  }, []);

  const handleItemClick = (msg) => {
    console.log(`Clicked: ${msg}`);
  };

  return (
    <div className="relative w-full min-h-screen bg-[#050A18] text-white overflow-x-hidden font-sans selection:bg-purple-500 selection:text-white">
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@100;300;400;500;700&display=swap');
        :root { --font-pingfang: 'PingFang SC', 'Noto Sans SC', sans-serif; }
        body { font-family: var(--font-pingfang); }
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #050A18; }
        ::-webkit-scrollbar-thumb { background: #1e293b; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #784BC5; }
        .glass-nav {
          background: rgba(5, 10, 24, 0.7);
          backdrop-filter: blur(20px);
          border-bottom: 1px solid rgba(255, 255, 255, 0.05);
        }
        .glass-card {
          background: rgba(255, 255, 255, 0.02);
          backdrop-filter: blur(12px);
          border: 1px solid rgba(255, 255, 255, 0.05);
          box-shadow: 0 4px 30px rgba(0, 0, 0, 0.2);
        }
        .glow-pulse { animation: pulse-glow 8s ease-in-out infinite; }
        @keyframes pulse-glow {
          0%, 100% { opacity: 0.3; transform: scale(1); }
          50% { opacity: 0.6; transform: scale(1.1); }
        }

        /* 彈窗樣式 */
        .modal-overlay {
          position: fixed; 
          z-index: 1000; 
          padding-top: 50px; 
          left: 0;
          top: 0;
          width: 100%; 
          height: 100%; 
          overflow: auto; 
          background-color: rgba(0,0,0,0.9); 
          display: flex;
          align-items: center;
          justify-content: center;
          animation: fadeIn 0.3s;
        }
        .modal-content {
          margin: auto;
          display: block;
          max-width: 90%;
          max-height: 90vh;
          object-fit: contain;
          animation: zoom 0.3s;
          border-radius: 8px;
          box-shadow: 0 0 50px rgba(255, 255, 255, 0.1);
        }
        @keyframes zoom {
          from {transform:scale(0.8); opacity: 0;} 
          to {transform:scale(1); opacity: 1;}
        }
        @keyframes fadeIn {
          from {opacity: 0;} 
          to {opacity: 1;}
        }
        .close-btn {
          position: absolute;
          top: 30px;
          right: 40px;
          color: #f1f1f1;
          font-size: 40px;
          font-weight: bold;
          transition: 0.3s;
          cursor: pointer;
          z-index: 1001;
        }
        .close-btn:hover { color: #bbb; }
      `}</style>

      {/* 背景光效 */}
      <div className="fixed inset-0 pointer-events-none z-0">
        <div className="absolute -top-[20%] left-[20%] w-[60%] h-[40%] rounded-full bg-[#0247F0] blur-[120px] opacity-20 glow-pulse"></div>
        <div className="absolute -bottom-[20%] right-[20%] w-[60%] h-[40%] rounded-full bg-[#784BC5] blur-[120px] opacity-20 glow-pulse" style={{ animationDelay: '4s' }}></div>
      </div>

      {/* 导航栏 */}
      <nav className={`fixed top-0 w-full z-50 transition-all duration-300 ${scrollY > 50 ? 'glass-nav py-3' : 'bg-transparent py-6'}`}>
        <div className="max-w-6xl mx-auto px-6 flex justify-between items-center">
          <div className="font-bold text-xl tracking-tight flex items-center gap-2 cursor-pointer" onClick={() => window.scrollTo({top:0, behavior:'smooth'})}>
            <div className="w-8 h-8 rounded bg-gradient-to-tr from-blue-600 to-purple-600 flex items-center justify-center">D</div>
            <span>DESIGN BY.LIUTINGTING·2025</span>
          </div>
          <div className="hidden md:flex gap-8 text-sm font-light">
            {[{ id: 'summary', label: '工作总结' }, { id: 'ai-experience', label: 'AI 经验' }, { id: 'future-plan', label: '未来规划' }].map(item => (
              <button 
                key={item.id}
                onClick={() => scrollToSection(item.id)}
                className={`transition-colors relative hover:text-white ${activeSection === item.id ? 'text-white font-medium' : 'text-gray-400'}`}
              >
                {item.label}
                {activeSection === item.id && <span className="absolute -bottom-2 left-0 w-full h-[2px] bg-purple-500 rounded-full animate-fade-in"></span>}
              </button>
            ))}
          </div>
        </div>
      </nav>

      {/* Hero 区域 */}
      <section className="relative w-full h-screen flex flex-col items-center justify-center z-10" id="home">
        <canvas ref={canvasRef} className="absolute inset-0 w-full h-full cursor-pointer" style={{ opacity: Math.max(0, 1 - scrollY / 700) }} title="点击屏幕试触发粒子冲击波" />
        <div className="relative z-20 text-center px-4 max-w-5xl mx-auto flex flex-col items-center gap-6 pointer-events-none" style={{ transform: `translateY(${scrollY * 0.5}px)`, opacity: Math.max(0, 1 - scrollY / 500) }}>
          <div className="glass-card px-3 py-1 rounded-full text-xs tracking-[0.2em] text-blue-300 uppercase pointer-events-auto">2025-2026 Annual Report</div>
          <h1 className="text-5xl md:text-8xl font-bold tracking-tighter text-transparent bg-clip-text bg-gradient-to-b from-white to-white/60 drop-shadow-lg pointer-events-auto">
            Personal<br />Annual Workshow
          </h1>
          <div className="h-8 text-xl md:text-2xl font-light text-purple-200 tracking-widest mt-2 flex items-center justify-center pointer-events-auto">
            <span className="mr-3 text-purple-400 opacity-60">///</span>
            <TypewriterText text="述职报告 · 创新与突破" delay={800} />
          </div>
          <div className="absolute bottom-10 left-1/2 -translate-x-1/2 animate-bounce opacity-50 pointer-events-auto cursor-pointer" onClick={() => scrollToSection('summary')}>
            <ChevronDown />
          </div>
        </div>
      </section>

      {/* 主体内容区域 */}
      <div className="relative z-20 max-w-5xl mx-auto px-6 pb-24 space-y-32">
        
        {/* 板块一：工作总结 */}
        <section id="summary" className="scroll-mt-32">
          <SectionTitle number="1" title="工作总结（核心：业务支持）" icon={Layout} />
          
          <div className="glass-card p-8 rounded-2xl md:col-span-2">
              <div className="flex justify-between items-start mb-8 border-b border-white/10 pb-6">
                <h3 className="text-2xl font-bold flex items-center gap-3">
                  <span className="w-2 h-8 bg-blue-500 rounded-full"></span>
                  企培业务
                </h3>
                <span className="bg-blue-500/20 text-blue-300 px-3 py-1 rounded-full text-xs font-mono">50% WORKLOAD</span>
              </div>
              
              <div className="grid grid-cols-1 lg:grid-cols-3 gap-8 pl-5 border-l border-white/10 ml-1">
                
                {/* 第一列：商业支持 (按月份复盘) */}
                <div className="flex flex-col gap-4">
                  <h4 className="text-lg text-blue-300 font-medium flex items-center gap-2">
                    <Zap size={16} /> 商业支持 · 重点项目
                  </h4>
                  <div className="space-y-3">
                     <ExpandableItem 
                        id="comm-jan"
                        title="一月 · 创新加速营" 
                        subtitle="一期+二期"
                        description="聚焦创新理念导入与项目孵化启动，完成全场景视觉支撑。"
                        activeId={expandedId}
                        onToggle={toggleExpand}
                        onZoom={openModal}
                      >
                        <p className="mb-2 text-blue-200 font-bold">兴火燎原创新加速营</p>
                        <p><span className="text-gray-400">定位：</span>开营阶段视觉支撑，严谨+活力。</p>
                        <p><span className="text-gray-400">产出：</span>现场物料100+，定制PPT课件4套。</p>
                        <p><span className="text-gray-400">价值：</span>视觉规整有序，提升教学效率。</p>
                      </ExpandableItem>

                      <ExpandableItem 
                        id="comm-feb"
                        title="二月 · 决赛冲刺坊" 
                        subtitle="答辩指导配套"
                        description="聚焦成果打磨，强化冲刺竞技氛围。"
                        activeId={expandedId}
                        onToggle={toggleExpand}
                        onZoom={openModal}
                      >
                         <p className="mb-2 text-blue-200 font-bold">兴火燎原决赛冲刺坊</p>
                         <p><span className="text-gray-400">定位：</span>保障答辩环节视觉专业性。</p>
                         <p><span className="text-gray-400">产出：</span>物料29+，学员手册(44P)，场景模拟图。</p>
                         <p><span className="text-gray-400">价值：</span>解决认知盲区，提升筹备效率。</p>
                      </ExpandableItem>

                      <ExpandableItem 
                        id="comm-mar"
                        title="三月 · 沙龙+赛事" 
                        subtitle="科技风+奖杯定制"
                        description="数金系列沙龙与赛事荣誉体系打造。"
                        activeId={expandedId}
                        onToggle={toggleExpand}
                        onZoom={openModal}
                      >
                        <p className="mb-2 text-blue-200 font-bold">数金沙龙 & 赛事配套</p>
                        <p><span className="text-gray-400">定位：</span>金融科技属性，凸显荣誉感。</p>
                        <p><span className="text-gray-400">产出：</span>沙龙物料18件，奖杯18个，模拟图12张。</p>
                        <p><span className="text-gray-400">价值：</span>精准匹配调性，强化品牌专属感。</p>
                      </ExpandableItem>

                      <ExpandableItem 
                        id="comm-apr"
                        title="四月 · 马拉松公开赛" 
                        subtitle="巅峰赛事"
                        description="全流程视觉与兴闪亮沙龙现场物料。"
                        activeId={expandedId}
                        onToggle={toggleExpand}
                        onZoom={openModal}
                      >
                        <p className="mb-2 text-blue-200 font-bold">马拉松公开赛 & 兴闪亮</p>
                        <p><span className="text-gray-400">定位：</span>竞技协作 + 轻松交流。</p>
                        <p><span className="text-gray-400">产出：</span>大场景物料64个，赛事手册(32P)，定制PPT。</p>
                        <p><span className="text-gray-400">价值：</span>保障多环节有序推进，获一致好评。</p>
                      </ExpandableItem>
                  </div>
                </div>

                {/* 第二列：商业海报 */}
                <div className="flex flex-col gap-4">
                  <h4 className="text-lg text-blue-300 font-medium flex items-center gap-2">
                    <Target size={16} /> 商业海报支持
                  </h4>
                  <div className="h-full">
                     <TechImagePlaceholder label="商业课程海报" subLabel="点击上传案例图" height="h-full min-h-[160px]" onZoom={openModal} />
                  </div>
                </div>

                {/* 第三列：UI 与 运营支持 */}
                <div className="flex flex-col gap-4">
                   <h4 className="text-lg text-blue-300 font-medium flex items-center gap-2">
                    <Layout size={16} /> UI 与 运营支持
                  </h4>
                  <div className="space-y-4">
                    <TechImagePlaceholder label="训练营海报" subLabel="点击上传案例图" onZoom={openModal} />
                    <TechImagePlaceholder label="UI 案例展示" subLabel="点击上传 APP 界面图" onZoom={openModal} />
                    {/* 广告展示位 */}
                    <TechImagePlaceholder label="广告位展示" subLabel="点击上传活动广告图" onZoom={openModal} />
                  </div>
                </div>

              </div>
          </div>

          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            {/* 1.2 线上课程 */}
            <div className="glass-card p-8 rounded-2xl">
              <div className="flex justify-between items-start mb-6">
                 <h3 className="text-xl font-bold">线上课程业务</h3>
                 <span className="bg-purple-500/20 text-purple-300 px-3 py-1 rounded-full text-xs font-mono">40%</span>
              </div>
              <div className="space-y-3">
                <ExpandableItem 
                  id="online-camp"
                  title="2.1 全年训练营" 
                  subtitle="视觉包装与物料迭代"
                  description="负责全年 12 期训练营的整体视觉包装与宣发物料迭代。"
                  activeId={expandedId}
                  onToggle={toggleExpand}
                  onZoom={openModal}
                />
                <ExpandableItem 
                  id="online-new"
                  title="2.2 新课开发" 
                  subtitle="教研配合与课件设计"
                  description="配合教研团队完成《高效能人士》新课 PPT 及逐字稿配图。"
                  activeId={expandedId}
                  onToggle={toggleExpand}
                  onZoom={openModal}
                />
                <ExpandableItem 
                  id="online-rec"
                  title="2.3 重点推介" 
                  subtitle="高质量课程长图"
                  description="每月 output 1-2 张高质量课程长图，转化率提升 15%。"
                  activeId={expandedId}
                  onToggle={toggleExpand}
                  onZoom={openModal}
                />
              </div>
            </div>

            {/* 1.3 其他业务 */}
            <div className="glass-card p-8 rounded-2xl flex flex-col justify-between">
               <div className="flex justify-between items-start mb-6">
                 <h3 className="text-xl font-bold">其他业务</h3>
                 <span className="bg-gray-500/20 text-gray-300 px-3 py-1 rounded-full text-xs font-mono">10%</span>
              </div>
              <div className="space-y-3">
                <ExpandableItem 
                  id="other-media"
                  title="媒体业务支持" 
                  subtitle="公众号与视频号"
                  description="公众号头图优化、视频号封面规范制定，统一品牌视觉语言。"
                  activeId={expandedId}
                  onToggle={toggleExpand}
                  onZoom={openModal}
                />
                <ExpandableItem 
                  id="other-hr"
                  title="人力行政" 
                  subtitle="企业文化建设"
                  description="企业文化墙更新、员工手册设计，提升内部凝聚力。"
                  activeId={expandedId}
                  onToggle={toggleExpand}
                  onZoom={openModal}
                />
                <ExpandableItem 
                  id="other-school"
                  title="学堂板块支持" 
                  subtitle="平台视觉与运营"
                  description="负责线上学堂平台的界面视觉优化、课程封面规范制定及日常运营活动图设计。"
                  activeId={expandedId}
                  onToggle={toggleExpand}
                  onZoom={openModal}
                />
              </div>
            </div>
          </div>
        </section>

        {/* 板块二：AI 经验 */}
        <section id="ai-experience" className="scroll-mt-32">
          <SectionTitle number="2" title="AI 经验与应用" icon={Cpu} />

          <div className="glass-card p-1 rounded-2xl overflow-hidden mb-8">
            <div className="bg-gradient-to-r from-blue-900/40 to-purple-900/40 p-8">
              <h3 className="text-2xl font-bold mb-6 flex items-center gap-3">
                <Award className="text-yellow-400" />
                重点成果：民生银行六格场景漫画
              </h3>
              <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 items-center">
                {/* 六格漫画 13 张 */}
                <TechImagePlaceholder label="六格漫画海报" subLabel="点击上传最终海报图" maxImages={13} onZoom={openModal} />
                <div className="space-y-8">
                   
                  {/* 可视化制作过程 */}
                  <div>
                    <h4 className="text-blue-300 font-medium mb-4 flex items-center gap-2">
                       <PenTool size={16} /> 制作过程 · AI 工作流
                    </h4>
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-4 relative">
                        {/* 装饰线 */}
                        <div className="hidden md:block absolute top-1/2 left-0 w-full h-[1px] bg-gradient-to-r from-transparent via-white/20 to-transparent -translate-y-1/2 z-0"></div>

                        {/* Step 1 */}
                        <div className="bg-white/5 border border-white/10 p-4 rounded-xl relative z-10 backdrop-blur-sm hover:bg-white/10 hover:border-blue-500/30 transition-all group">
                            <div className="w-8 h-8 rounded-full bg-blue-500/20 text-blue-300 flex items-center justify-center text-xs font-bold mb-3 border border-blue-500/30 group-hover:scale-110 transition-transform">01</div>
                            <h5 className="text-white text-sm font-bold mb-1 flex items-center gap-2">
                               <Zap size={12} className="text-yellow-400" /> 灵感草图
                            </h5>
                            <p className="text-xs text-gray-400 leading-relaxed">
                               使用 <span className="text-blue-300 font-bold">Midjourney</span> 快速生成分镜草图，确定构图视角与光影基调。
                            </p>
                        </div>

                        {/* Step 2 */}
                        <div className="bg-white/5 border border-white/10 p-4 rounded-xl relative z-10 backdrop-blur-sm hover:bg-white/10 hover:border-purple-500/30 transition-all group">
                            <div className="w-8 h-8 rounded-full bg-purple-500/20 text-purple-300 flex items-center justify-center text-xs font-bold mb-3 border border-purple-500/30 group-hover:scale-110 transition-transform">02</div>
                            <h5 className="text-white text-sm font-bold mb-1 flex items-center gap-2">
                               <Layers size={12} className="text-purple-400" /> 角色控制
                            </h5>
                            <p className="text-xs text-gray-400 leading-relaxed">
                               利用 <span className="text-purple-300 font-bold">ControlNet</span> 提取骨架与边缘，锁定角色特征，確保多格畫面統一。
                            </p>
                        </div>

                        {/* Step 3 */}
                        <div className="bg-white/5 border border-white/10 p-4 rounded-xl relative z-10 backdrop-blur-sm hover:bg-white/10 hover:border-green-500/30 transition-all group">
                            <div className="w-8 h-8 rounded-full bg-green-500/20 text-green-300 flex items-center justify-center text-xs font-bold mb-3 border border-green-500/30 group-hover:scale-110 transition-transform">03</div>
                            <h5 className="text-white text-sm font-bold mb-1 flex items-center gap-2">
                               <Monitor size={12} className="text-green-400" /> 排版精修
                            </h5>
                            <p className="text-xs text-gray-400 leading-relaxed">
                               導入 <span className="text-green-300 font-bold">Photoshop</span> 進行分鏡排版、對話氣泡植入及細節光影人工精修。
                            </p>
                        </div>
                    </div>
                  </div>

                  <div>
                     <h4 className="text-blue-300 font-medium mb-2">成果亮点</h4>
                     <div className="flex gap-4">
                       <div className="text-center p-3 bg-white/5 rounded-lg flex-1 hover:bg-white/10 cursor-pointer transition-colors" onClick={() => alert('详细数据分析报告待补充')}>
                         <div className="text-2xl font-bold text-white">-60%</div>
                         <div className="text-xs text-gray-500">制作周期</div>
                       </div>
                       <div className="text-center p-3 bg-white/5 rounded-lg flex-1 hover:bg-white/10 cursor-pointer transition-colors" onClick={() => alert('客户反馈记录待补充')}>
                         <div className="text-2xl font-bold text-white">Top 1</div>
                         <div className="text-xs text-gray-500">客户满意度</div>
                       </div>
                     </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div className="grid grid-cols-2 md:grid-cols-3 gap-6">
             <div className="col-span-2 md:col-span-1 glass-card p-6 rounded-2xl flex flex-col justify-center hover:bg-white/5 transition-colors cursor-pointer" onClick={() => alert('正在加载更多案例...')}>
               <h3 className="text-xl font-bold mb-2">其他 AI 产出</h3>
               <p className="text-sm text-gray-400 mb-4">将 AI 融入日常海报设计，提升素材丰富度。</p>
               <button className="text-sm text-blue-400 flex items-center gap-1 hover:gap-2 transition-all">
                 查看更多 <ArrowRight size={14} />
               </button>
             </div>
             <TechImagePlaceholder label="课程推荐海报 01" subLabel="点击上传" onZoom={openModal} />
             <TechImagePlaceholder label="课程推荐海报 02" subLabel="点击上传" onZoom={openModal} />
          </div>
        </section>

        {/* 板块三：未来规划 */}
        <section id="future-plan" className="scroll-mt-32">
          <SectionTitle number="3" title="未来规划" icon={BarChart2} />
          
          <div className="glass-card p-1 rounded-2xl relative overflow-hidden group">
            <div className="absolute top-0 right-0 w-64 h-64 bg-blue-500/10 rounded-full blur-[80px] group-hover:bg-blue-500/20 transition-colors"></div>
            
            <div className="p-10 relative z-10">
              <div className="flex flex-col md:flex-row gap-10">
                 <div className="md:w-1/3">
                   <h3 className="text-4xl font-bold mb-4 bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-purple-400">2026<br/>Goals</h3>
                   <p className="text-gray-400 text-sm">聚焦两大核心方向，实现设计价值突破。</p>
                 </div>
                 
                 <div className="md:w-2/3 space-y-6">
                   {[
                     { 
                       title: "深耕现有业务板块，筑牢服务根基", 
                       desc: "持续聚焦商业项目、SaaS 业务线、市场品宣、产品 UI 四大核心支持维度，精准匹配各业务场景的设计需求，强化金融、科技类高端项目的视觉质感与品牌辨识度，保障项目交付质量与效率，深化与核心客户的合作黏性。" 
                     },
                     { 
                       title: "深化 AI 技术学习与应用，赋能提质增效", 
                       desc: "系统学习 AI 设计的进阶应用方法，沉淀不同业务场景的 AI 脚本撰写、咒语优化、素材生成的标准化方法论；拓展 AI 在视觉创意、批量物料制作、交互原型设计等场景的应用边界，通过“AI 辅助 + 人工精修”模式，进一步压缩项目工期、丰富设计风格，实现设计工作的效率与质量双提升。" 
                     }
                   ].map((plan, i) => (
                     <div 
                        key={i} 
                        className="flex gap-4 group/item cursor-pointer hover:bg-white/5 p-5 rounded-xl transition-all border border-transparent hover:border-white/5"
                        onClick={() => handleItemClick(`未来规划-${plan.title}`)}
                     >
                       <div className="shrink-0 w-12 h-12 rounded-full border border-white/10 flex items-center justify-center text-lg font-mono text-gray-500 group-hover/item:text-white group-hover/item:bg-blue-600 group-hover/item:border-transparent transition-all mt-1">
                         0{i+1}
                       </div>
                       <div>
                         <h4 className="text-xl font-bold text-white mb-3 group-hover/item:text-blue-300 transition-colors flex items-center gap-2">
                             {plan.title}
                             <CheckCircle size={16} className="opacity-0 group-hover/item:opacity-100 transition-opacity text-green-400"/>
                         </h4>
                         <p className="text-gray-400 text-sm leading-relaxed text-justify">{plan.desc}</p>
                       </div>
                     </div>
                   ))}
                 </div>
              </div>
            </div>
          </div>
        </section>

      </div>
      
      {/* 弹窗 DOM */}
      {modalData.isOpen && (
        <div className="modal-overlay" onClick={closeModal}>
          <span className="close-btn" onClick={closeModal}>&times;</span>
          <img className="modal-content" src={modalData.src} alt={modalData.alt} onClick={(e) => e.stopPropagation()} />
          {modalData.alt && <div className="absolute bottom-10 left-0 right-0 text-center text-white/80 bg-black/50 p-2 pointer-events-none">{modalData.alt}</div>}
        </div>
      )}

      <footer className="relative z-10 py-12 text-center text-gray-600 text-xs border-t border-white/5">
        <p className="mb-2">&copy; 2026 Personal Annual Workshow. </p>
        <p className="opacity-50">Powered by React & Native Canvas 3D Engine.</p>
      </footer>
    </div>
  );
}
