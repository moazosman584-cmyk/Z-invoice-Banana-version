<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فاتورة ضريبية وسند قبض - مع مساعد الأصناف الذكي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcode-generator/1.4.4/qrcode.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;500;600;700;800;900&display=swap');
        
        * {
            font-family: 'Cairo', sans-serif;
            box-sizing: border-box;
        }

        @media print {
            .no-print, .stamp-controls, .stamp-btn-resize {
                display: none !important;
                opacity: 0 !important;
            }
            body {
                background: white !important;
                padding: 0 !important;
                margin: 0 !important;
                -webkit-print-color-adjust: exact !important;
                print-color-adjust: exact !important;
            }
            .page-border-wrapper {
                padding: 15px !important;
                margin: 0 !important;
                box-shadow: none !important;
                max-width: 100% !important;
                page-break-inside: avoid !important;
                position: relative !important;
            }
            .tpl-1 { border: 2px solid #1e293b !important; border-radius: 8px !important; }
            .tpl-2 { border: 2px solid #0f766e !important; border-top: 8px solid #0f766e !important; border-radius: 8px !important; }
            .tpl-3 { border: 2px solid #0096d6 !important; border-top: 8px solid #0096d6 !important; border-radius: 8px !important; }
            .tpl-4 { border: 1px solid #cbd5e1 !important; border-radius: 12px !important; }
            .receipt-border { border: 2px solid #1e293b !important; border-radius: 8px !important; }
            
            .draggable-stamp-container {
                position: absolute !important;
                -webkit-print-color-adjust: exact !important;
                print-color-adjust: exact !important;
            }

            @page {
                size: A4 portrait;
                margin: 8mm;
            }
        }

        .editable-field {
            transition: all 0.2s;
            min-width: 20px;
            display: inline-block;
        }
        .editable-field:hover, .editable-field:focus {
            background-color: rgba(241, 245, 249, 0.3);
            outline: 2px solid #2563eb;
            border-radius: 4px;
            padding: 1px 4px;
        }

        .draggable-stamp-container {
            position: absolute;
            cursor: move;
            user-select: none;
            z-index: 50;
            display: inline-block;
            padding: 6px;
            touch-action: none;
        }

        .draggable-stamp-container .stamp-controls,
        .draggable-stamp-container .stamp-btn-resize {
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.2s ease-in-out;
        }

        .draggable-stamp-container:hover .stamp-controls,
        .draggable-stamp-container:hover .stamp-btn-resize,
        .draggable-stamp-container.active-stamp .stamp-controls,
        .draggable-stamp-container.active-stamp .stamp-btn-resize {
            opacity: 1;
            pointer-events: auto;
        }

        .stamp-controls {
            position: absolute;
            top: -15px;
            left: 0;
            right: 0;
            display: flex;
            justify-content: center;
            gap: 4px;
            z-index: 60;
        }

        .stamp-ctrl-btn {
            background: #2563eb;
            color: white;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 11px;
            cursor: pointer;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
            transition: transform 0.1s;
        }
        .stamp-ctrl-btn:hover { transform: scale(1.1); }
        .stamp-btn-delete { background: #ef4444; }
        .stamp-btn-save { background: #059669; }
        .stamp-btn-rotate { cursor: grab; }
        .stamp-btn-resize {
            position: absolute;
            bottom: -6px;
            right: -6px;
            background: #2563eb;
            color: white;
            border-radius: 3px;
            width: 16px;
            height: 16px;
            cursor: nwse-resize;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
            z-index: 60;
        }

        /* TEMPLATE STYLES */
        .tpl-1 { border: 3px double #1e293b; border-radius: 16px; background-color: #ffffff; box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05); }
        .tpl-1 #headerSection { flex-direction: row; }
        .tpl-1 .seller-box { background-color: rgba(254, 242, 242, 0.4); border: 1px solid #fee2e2; border-radius: 0.75rem; }
        .tpl-1 .seller-header { color: #b91c1c; border-bottom: 1px solid rgba(254, 202, 202, 0.6); }
        .tpl-1 .seller-save-btn { background-color: #dc2626; color: #ffffff; }
        .tpl-1 .seller-save-btn:hover { background-color: #b91c1c; }
        .tpl-1 .buyer-box { background-color: rgba(239, 246, 255, 0.4); border: 1px solid #dbeafe; border-radius: 0.75rem; }
        .tpl-1 .buyer-header { color: #1d4ed8; border-bottom: 1px solid rgba(191, 219, 254, 0.6); }
        .tpl-1 .buyer-save-btn { background-color: #2563eb; color: #ffffff; }
        .tpl-1 .buyer-save-btn:hover { background-color: #1d4ed8; }
        .tpl-1 .table-hdr { background-color: #1e293b; color: #ffffff; }
        .tpl-1 .totals-card { background-color: #0f172a; color: #ffffff; }
        .tpl-1 .totals-grand-text { color: #fbbf24; border-top: 1px solid #334155; }
        .tpl-1 .badge-type { color: #dc2626; background-color: #fef2f2; border: 1px solid #fee2e2; }

        .tpl-2 { border: 2px solid #0f766e; border-top: 10px solid #0f766e; border-radius: 12px; background-color: #ffffff; box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05); }
        .tpl-2 #headerSection { flex-direction: row; }
        .tpl-2 .metadata-container { background-color: #f8fafc; border: 1px solid #e2e8f0; padding: 10px 14px; border-radius: 12px; box-shadow: inset 0 1px 2px rgba(0,0,0,0.02); }
        .tpl-2 .seller-box { background-color: #f0fdf4; border: 1px solid #bbf7d0; border-right: 5px solid #059669; border-radius: 0.75rem; }
        .tpl-2 .seller-header { color: #047857; border-bottom: 1px solid #a7f3d0; }
        .tpl-2 .seller-save-btn { background-color: #059669; color: #ffffff; }
        .tpl-2 .buyer-box { background-color: #f0fdfa; border: 1px solid #99f6e4; border-right: 5px solid #0d9488; border-radius: 0.75rem; }
        .tpl-2 .buyer-header { color: #0f766e; border-bottom: 1px solid #99f6e4; }
        .tpl-2 .buyer-save-btn { background-color: #0d9488; color: #ffffff; }
        .tpl-2 .table-hdr { background-color: #0f766e; color: #ffffff; }
        .tpl-2 .totals-card { background-color: #064e3b; color: #ffffff; border: 1px solid #047857; }
        .tpl-2 .totals-grand-text { color: #fde047; border-top: 1px solid #047857; }
        .tpl-2 .badge-type { color: #0d9488; background-color: #ccfbf1; border: 1px solid #99f6e4; }

        .tpl-3 { border: 2px solid #0096d6; border-top: 10px solid #0096d6; border-radius: 12px; background-color: #ffffff; box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05); }
        .tpl-3 #headerSection { background-color: #0096d6; color: #ffffff; padding: 16px 20px; border-radius: 12px; border-bottom: none; display: flex !important; flex-direction: row !important; justify-content: space-between !important; align-items: center !important; direction: rtl !important; }
        .tpl-3 #headerSection > div:first-child { order: 1 !important; }
        .tpl-3 #headerSection > div:last-child { order: 2 !important; }
        .tpl-3 .metadata-container { background-color: rgba(255, 255, 255, 0.2); border: 1px solid rgba(255, 255, 255, 0.35); padding: 10px 16px; border-radius: 12px; color: #ffffff; }
        .tpl-3 .metadata-container span { color: #ffffff !important; }
        .tpl-3 .badge-type { color: #ffffff; background-color: transparent; border: none; font-size: 1.3rem; font-weight: 800; padding: 0; }
        .tpl-3 .seller-box { background-color: #f0f8fd; border: 1px solid #d2e8f7; border-radius: 0.75rem; }
        .tpl-3 .seller-header { color: #0096d6; border-bottom: 1px solid #d2e8f7; }
        .tpl-3 .seller-save-btn { background-color: #0096d6; color: #ffffff; }
        .tpl-3 .buyer-box { background-color: #f0f8fd; border: 1px solid #d2e8f7; border-radius: 0.75rem; }
        .tpl-3 .buyer-header { color: #0096d6; border-bottom: 1px solid #d2e8f7; }
        .tpl-3 .buyer-save-btn { background-color: #0096d6; color: #ffffff; }
        .tpl-3 .table-hdr { background-color: #0096d6; color: #ffffff; }
        .tpl-3 .totals-card { background-color: #f0f8fd; color: #1e293b; border: 1px solid #d2e8f7; }
        .tpl-3 .totals-card .text-slate-300 { color: #475569 !important; }
        .tpl-3 .totals-grand-text { color: #0096d6; border-top: 1px solid #c0e0f5; }

        .tpl-4 { border: 1px solid #e2e8f0; border-radius: 16px; background-color: #ffffff; box-shadow: 0 4px 20px -2px rgba(0, 0, 0, 0.05); }
        .tpl-4 #headerSection { display: grid !important; grid-template-columns: repeat(3, minmax(0, 1fr)) !important; align-items: center !important; padding-bottom: 1.5rem !important; border-bottom: 1px solid #f1f5f9 !important; gap: 1rem !important; background-color: transparent !important; }
        .tpl-4 #headerSection > div:first-child { order: 1 !important; justify-content: flex-start !important; }
        .tpl-4 #tpl4TitleBlock { order: 2 !important; display: flex !important; flex-direction: column !important; align-items: center !important; justify-content: center !important; }
        .tpl-4 #headerSection > div:last-child { order: 3 !important; display: flex !important; flex-direction: column !important; align-items: flex-end !important; }
        .tpl-4 .metadata-container { background-color: transparent !important; border: none !important; padding: 0 !important; box-shadow: none !important; }
        .tpl-4 .badge-type { display: none !important; }
        .tpl-4 .seller-box, .tpl-4 .buyer-box { background-color: #f8fafc !important; border: 1px solid #e2e8f0 !important; border-radius: 12px !important; }
        .tpl-4 .seller-header { color: #1e293b !important; border-bottom: 1px solid rgba(226, 232, 240, 0.6) !important; }
        .tpl-4 .seller-save-btn { background-color: #0284c7 !important; color: #ffffff !important; }
        .tpl-4 .buyer-header { color: #1e293b !important; border-bottom: 1px solid rgba(226, 232, 240, 0.6) !important; }
        .tpl-4 .buyer-save-btn { background-color: #0284c7 !important; color: #ffffff !important; }
        .tpl-4 .table-hdr { background-color: #0284c7 !important; color: #ffffff !important; }
        .tpl-4 .totals-card { background-color: #f8fafc !important; color: #475569 !important; border: 1px solid #e2e8f0 !important; }
        .tpl-4 .totals-card .text-slate-300 { color: #64748b !important; }
        .tpl-4 .totals-grand-text { color: #0284c7 !important; font-weight: 800 !important; border-top: 1px solid #e2e8f0 !important; }
        .tpl-4 #qrcodeContainer { width: 125px !important; height: 125px !important; min-width: 125px !important; min-height: 125px !important; }
        .tpl-4 #qrCanvas { width: 115px !important; height: 115px !important; }
    </style>
</head>
<body class="bg-slate-100 min-h-screen p-2 sm:p-6">

    <div id="loadingOverlay" class="fixed inset-0 bg-black/60 z-[100] flex flex-col items-center justify-center text-white hidden">
        <i class="fa-solid fa-spinner fa-spin text-4xl mb-3 text-blue-400"></i>
        <p class="font-bold text-sm" id="loadingText">جاري المعالجة...</p>
    </div>

    <div class="max-w-4xl mx-auto">
        
        <div class="no-print bg-white p-3 rounded-2xl shadow-sm border border-slate-200 mb-4 flex flex-wrap justify-between items-center gap-3">
            <div>
                <h1 class="text-sm font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-file-invoice text-blue-600"></i>
                    محرر الفاتورة والسند (مزود بالمساعد الذكي)
                </h1>
                <p class="text-[11px] text-slate-500">تعديل التسلسل لحظياً بمجرد كتابة وتغيير التاريخ</p>
            </div>
            
            <div class="flex flex-wrap items-center gap-2">
                <!-- زر المساعد الذكي -->
                <button onclick="openAiModal()" class="bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-700 hover:to-indigo-700 text-white text-xs font-bold px-3 py-2 rounded-xl shadow-md transition flex items-center gap-1.5 animate-pulse">
                    <i class="fa-solid fa-wand-magic-sparkles text-amber-300"></i>
                    مساعد الأصناف والذكاء الاصطناعي
                </button>

                <button onclick="openNewInvoiceTab()" class="bg-blue-600 hover:bg-blue-700 text-white text-xs font-bold px-3 py-2 rounded-xl shadow-sm transition flex items-center gap-1.5">
                    <i class="fa-solid fa-square-plus"></i>
                    إنشاء فاتورة جديدة
                </button>

                <button onclick="saveCurrentInvoice()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold px-3 py-2 rounded-xl shadow-sm transition flex items-center gap-1.5">
                    <i class="fa-solid fa-box-archive"></i>
                    حفظ الفاتورة محلياً
                </button>
                <button onclick="openArchiveModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold px-3 py-2 rounded-xl shadow-sm transition flex items-center gap-1.5">
                    <i class="fa-solid fa-folder-open"></i>
                    أرشيف الفواتير
                </button>

                <div class="flex items-center gap-1.5 bg-amber-50 p-1 rounded-xl border border-amber-200">
                    <button id="btnViewInvoice" onclick="switchView('invoice')" class="bg-slate-800 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5">
                        <i class="fa-solid fa-file-invoice"></i>
                        الفاتورة
                    </button>
                    <button id="btnViewReceipt" onclick="switchView('receipt')" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5">
                        <i class="fa-solid fa-receipt text-amber-600"></i>
                        سند القبض
                    </button>
                </div>

                <div id="templateControls" class="flex items-center gap-1.5 bg-slate-100 p-1 rounded-xl border border-slate-200">
                    <button id="btnTpl1" onclick="switchTemplate(1)" class="bg-slate-800 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5">
                        <i class="fa-solid fa-layer-group"></i>
                        قالب 1
                    </button>
                    <button id="btnTpl2" onclick="switchTemplate(2)" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5">
                        <i class="fa-solid fa-palette text-teal-600"></i>
                        قالب 2
                    </button>
                    <button id="btnTpl3" onclick="switchTemplate(3)" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5">
                        <i class="fa-solid fa-gem text-sky-600"></i>
                        قالب 3
                    </button>
                    <button id="btnTpl4" onclick="switchTemplate(4)" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5">
                        <i class="fa-solid fa-crown text-sky-500"></i>
                        قالب 4
                    </button>
                </div>

                <label class="bg-amber-100 hover:bg-amber-200 text-amber-900 text-xs font-bold px-3 py-2 rounded-xl cursor-pointer border border-amber-300 transition flex items-center gap-1.5">
                    <i class="fa-solid fa-stamp text-amber-600"></i>
                    ختم السند
                    <input type="file" id="receiptStampUpload" accept="image/*" class="hidden">
                </label>

                <label class="bg-emerald-100 hover:bg-emerald-200 text-emerald-900 text-xs font-bold px-3 py-2 rounded-xl cursor-pointer border border-emerald-300 transition flex items-center gap-1.5">
                    <i class="fa-solid fa-stamp text-emerald-600"></i>
                    ختم الفاتورة
                    <input type="file" id="invoiceStampUpload" accept="image/*" class="hidden">
                </label>

                <button onclick="openModal('stamp')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-xl border transition flex items-center gap-1.5">
                    <i class="fa-solid fa-stamp text-emerald-600"></i>
                    فهرس الأختام
                </button>

                <label class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-xl cursor-pointer border transition flex items-center gap-1.5">
                    <i class="fa-solid fa-image text-blue-600"></i>
                    تغيير الشعار
                    <input type="file" id="logoUpload" accept="image/*" class="hidden">
                </label>

                <button onclick="openModal('logo')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-xl border transition flex items-center gap-1.5">
                    <i class="fa-solid fa-images text-amber-600"></i>
                    فهرس الشعارات
                </button>

                <button onclick="triggerGenerateQR()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold px-3 py-2 rounded-xl border transition flex items-center gap-1.5 shadow-sm">
                    <i class="fa-solid fa-qrcode"></i>
                    توليد QR
                </button>

                <button onclick="openModal('seller')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-xl border transition flex items-center gap-1.5">
                    <i class="fa-solid fa-building text-red-600"></i>
                    فهرس البائعين
                </button>

                <button onclick="openModal('buyer')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-xl border transition flex items-center gap-1.5">
                    <i class="fa-solid fa-users text-blue-600"></i>
                    فهرس العملاء
                </button>

                <button onclick="window.print()" class="bg-red-600 hover:bg-red-700 text-white text-xs font-bold px-4 py-2 rounded-xl shadow-sm transition flex items-center gap-1.5">
                    <i class="fa-solid fa-print"></i>
                    طباعة / حفظ PDF
                </button>
            </div>
        </div>

        <!-- MAIN INVOICE -->
        <div id="invoiceView">
            <div id="invoiceWrapper" class="page-border-wrapper tpl-1 p-5 sm:p-8 relative min-h-[750px]">
                
                <div id="invoiceStampContainer" class="draggable-stamp-container hidden" style="bottom: 80px; left: 80px;">
                    <div class="stamp-controls">
                        <div class="stamp-ctrl-btn stamp-btn-save" onclick="saveStampFromContainer('invoiceStampContainer')" title="حفظ الختم في الفهرس المحلي"><i class="fa-solid fa-floppy-disk text-[10px]"></i></div>
                        <div class="stamp-ctrl-btn stamp-btn-rotate" title="تدوير الختم"><i class="fa-solid fa-rotate text-[10px]"></i></div>
                        <div class="stamp-ctrl-btn stamp-btn-delete" onclick="removeStamp('invoiceStampContainer')" title="حذف الختم"><i class="fa-solid fa-xmark text-[10px]"></i></div>
                    </div>
                    <img id="invoiceStampImg" src="" alt="ختم الفاتورة" class="w-32 h-32 object-contain opacity-95 pointer-events-none">
                    <div class="stamp-btn-resize" title="تكبير/تصغير الختم"></div>
                </div>

                <div id="headerSection" class="flex justify-between items-center border-b-2 border-slate-200 pb-4 mb-4">
                    <div class="flex items-center gap-3">
                        <div class="flex flex-col items-center gap-1.5">
                            <div class="w-20 h-20 border border-slate-200 rounded-xl flex items-center justify-center p-1 bg-white shrink-0 overflow-hidden shadow-sm">
                                <img id="companyLogo" src="https://via.placeholder.com/150?text=الشعار" alt="شعار المؤسسة" class="max-w-full max-h-full object-contain">
                            </div>
                            <button onclick="saveLogo()" class="no-print bg-slate-200 hover:bg-slate-300 text-slate-700 text-[10px] font-bold px-2 py-0.5 rounded transition flex items-center gap-1">
                                <i class="fa-solid fa-floppy-disk"></i> حفظ
                            </button>
                        </div>

                        <div class="text-right">
                            <div>
                                <span class="badge-type inline-block text-xs font-bold px-2.5 py-0.5 rounded-full mt-1">فاتورة ضريبية</span>
                            </div>
                        </div>
                    </div>

                    <div id="tpl4TitleBlock" class="hidden text-center space-y-1">
                        <h2 class="text-xl font-black text-[#0284c7]">فاتورة ضريبية</h2>
                        <div id="tpl4MetaCenter" class="hidden text-xs text-slate-700 font-normal space-y-0.5 pt-1">
                            <div>رقم الفاتورة: <span class="font-mono editable-field" contenteditable="true" id="tpl4InvNum">INV-2355</span></div>
                            <div>التاريخ: <span class="font-mono editable-field" contenteditable="true" id="tpl4InvDate">12/08/2026</span></div>
                        </div>
                    </div>

                    <div class="text-right space-y-1 text-xs sm:text-sm metadata-container" id="standardMetaContainer">
                        <div class="flex items-center gap-2 justify-end" id="stdInvNumRow">
                            <span class="font-normal text-slate-500">رقم الفاتورة:</span> 
                            <span class="font-mono font-normal editable-field" contenteditable="true" id="stdInvNum">INV-2355</span>
                        </div>
                        <div class="flex items-center gap-2 justify-end" id="stdInvDateRow">
                            <span class="font-normal text-slate-500">التاريخ:</span> 
                            <span class="font-mono font-normal editable-field" contenteditable="true" id="stdInvDate">12/08/2026</span>
                        </div>
                        <div id="tpl4QrSlot" class="hidden pt-1 flex-col items-center">
                            <p class="text-[10px] font-bold text-slate-700 mt-1 text-center">رمز التحقق الضريبي - ZATCA</p>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-2 gap-4 mb-5 text-xs">
                    <div class="seller-box p-3.5 relative">
                        <div class="seller-header flex items-center justify-between font-bold text-xs mb-2 pb-1">
                            <div class="flex items-center gap-1.5"><span>بيانات البائع</span></div>
                            <div class="flex items-center gap-1 no-print">
                                <button onclick="addCustomField('sellerFields')" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-[10px] font-bold px-2 py-0.5 rounded transition flex items-center gap-1">
                                    <i class="fa-solid fa-plus text-blue-600"></i> حقل
                                </button>
                                <button onclick="saveCurrentParty('seller')" class="seller-save-btn text-[10px] font-bold px-2 py-0.5 rounded transition flex items-center gap-1">
                                    <i class="fa-solid fa-floppy-disk"></i> حفظ
                                </button>
                            </div>
                        </div>
                        <div class="space-y-1" id="sellerFields">
                            <p class="flex justify-between items-center"><span class="text-slate-500">الاسم:</span> <span class="font-normal text-slate-800 editable-field" contenteditable="true" id="sellerName">البائع</span></p>
                            <p class="flex justify-between items-center"><span class="text-slate-500">الرقم الضريبي:</span> <span class="font-mono font-normal text-slate-800 editable-field" contenteditable="true" id="sellerVat">-</span></p>
                            <p class="flex justify-between items-center"><span class="text-slate-500">السجل التجاري:</span> <span class="font-mono font-normal text-slate-800 editable-field" contenteditable="true" id="sellerCr">-</span></p>
                            <p class="flex justify-between items-center"><span class="text-slate-500">العنوان:</span> <span class="font-normal text-slate-800 editable-field" contenteditable="true" id="sellerAddress">-</span></p>
                        </div>
                    </div>

                    <div class="buyer-box p-3.5 relative">
                        <div class="buyer-header flex items-center justify-between font-bold text-xs mb-2 pb-1">
                            <div class="flex items-center gap-1.5"><span>بيانات العميل</span></div>
                            <div class="flex items-center gap-1 no-print">
                                <button onclick="addCustomField('buyerFields')" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-[10px] font-bold px-2 py-0.5 rounded transition flex items-center gap-1">
                                    <i class="fa-solid fa-plus text-blue-600"></i> حقل
                                </button>
                                <button onclick="saveCurrentParty('buyer')" class="buyer-save-btn text-[10px] font-bold px-2 py-0.5 rounded transition flex items-center gap-1">
                                    <i class="fa-solid fa-floppy-disk"></i> حفظ
                                </button>
                            </div>
                        </div>
                        <div class="space-y-1" id="buyerFields">
                            <p class="flex justify-between items-center"><span class="text-slate-500">الاسم:</span> <span class="font-normal text-slate-800 editable-field" contenteditable="true" id="buyerName">عميل نقدي</span></p>
                            <p class="flex justify-between items-center"><span class="text-slate-500">الرقم الضريبي:</span> <span class="font-mono font-normal text-slate-800 editable-field" contenteditable="true" id="buyerVat">-</span></p>
                            <p class="flex justify-between items-center"><span class="text-slate-500">السجل التجاري:</span> <span class="font-mono font-normal text-slate-800 editable-field" contenteditable="true" id="buyerCr">-</span></p>
                            <p class="flex justify-between items-center"><span class="text-slate-500">العنوان:</span> <span class="font-normal text-slate-800 editable-field" contenteditable="true" id="buyerAddress">-</span></p>
                        </div>
                    </div>
                </div>

                <div class="overflow-x-auto mb-3 rounded-lg border border-slate-200">
                    <table class="w-full text-right text-xs border-collapse">
                        <thead>
                            <tr class="table-hdr font-bold">
                                <th class="p-2.5 w-[40%]">المنتج / الخدمة</th>
                                <th class="p-2.5 text-center w-[10%]">الكمية</th>
                                <th class="p-2.5 text-center w-[12%]">سعر الوحدة</th>
                                <th class="p-2.5 text-center w-[12%]">المجموع</th>
                                <th class="p-2.5 text-center w-[11%]">ضريبة (15%)</th>
                                <th class="p-2.5 text-center w-[12%]">الإجمالي</th>
                                <th class="p-1 text-center no-print w-[3%]">إجراء</th>
                            </tr>
                        </thead>
                        <tbody id="itemsBody" class="divide-y divide-slate-200 bg-white">
                        </tbody>
                    </table>
                </div>

                <!-- أزرار العمليات لجدول الأصناف -->
                <div class="no-print mb-4 flex items-center justify-between">
                    <button onclick="addNewRow()" class="bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold text-xs px-3 py-1.5 rounded-lg border border-slate-300 transition flex items-center gap-1.5">
                        <i class="fa-solid fa-plus text-blue-600"></i> إضافة منتج جديد
                    </button>

                    <!-- زرار ملء الجدول العشوائي -->
                    <button onclick="fillInvoiceWithAiPool()" class="bg-gradient-to-r from-amber-500 to-orange-600 hover:from-amber-600 hover:to-orange-700 text-white font-bold text-xs px-4 py-1.5 rounded-lg shadow-sm transition flex items-center gap-1.5">
                        <i class="fa-solid fa-dice text-amber-200"></i> ملء الجدول عشوائياً بالأصناف
                    </button>
                </div>

                <div id="summarySection" class="grid grid-cols-2 gap-4 items-center border-t border-slate-200 pt-3 mb-3">
                    <div class="text-xs text-slate-600 space-y-2">
                        <div>
                            <p class="font-normal text-slate-800">طريقة الدفع: <span id="paymentMethod" class="text-blue-700 editable-field" contenteditable="true">نقداً</span></p>
                            <p class="text-[10px] text-slate-400 mt-0.5">شكراً لتعاملكم معنا. تُعد هذه الفاتورة وثيقة رسمية.</p>
                        </div>
                        <div id="tpl2QrSection" class="hidden flex-col items-start pt-1">
                            <div id="tpl2QrSlot"></div>
                            <p class="text-[10px] font-bold text-slate-600 mt-1">رمز التحقق الضريبي (ZATCA Phase 2)</p>
                        </div>
                    </div>

                    <div class="totals-card p-3.5 rounded-xl space-y-1.5 text-xs">
                        <div class="flex justify-between text-slate-300">
                            <span>المجموع قبل الضريبة:</span>
                            <span id="subTotalDisplay" class="font-normal font-mono">0.00 ر.س</span>
                        </div>
                        <div class="flex justify-between text-slate-300">
                            <span>ضريبة القيمة المضافة (15%):</span>
                            <span id="vatTotalDisplay" class="font-normal font-mono">0.00 ر.س</span>
                        </div>
                        <div class="totals-grand-text flex justify-between font-bold text-sm pt-1.5">
                            <span>الإجمالي شامل الضريبة:</span>
                            <span id="grandTotalDisplay" class="font-mono">0.00 ر.س</span>
                        </div>

                        <div class="pt-2 space-y-1 border-t border-slate-300/30">
                            <div class="flex justify-between items-center text-slate-600">
                                <span>المبلغ المسدد:</span>
                                <span id="paidDisplay" class="font-normal font-mono editable-field" contenteditable="true">0.00 ر.س</span>
                            </div>
                            <div class="flex justify-between items-center text-slate-600">
                                <span>المبلغ المستحق:</span>
                                <span id="dueDisplay" class="font-normal font-mono editable-field" contenteditable="true">0.00 ر.س</span>
                            </div>
                        </div>
                    </div>
                </div>

                <div id="tpl1QrSection" class="flex flex-col items-center justify-center pt-2 border-t border-dashed border-slate-200">
                    <div id="tpl1QrSlot">
                        <div id="qrcodeContainer" class="p-2 bg-white border border-slate-300 rounded-xl shadow-sm flex items-center justify-center min-w-[120px] min-h-[120px] w-[120px] h-[120px]">
                            <canvas id="qrCanvas" class="max-w-full h-auto block"></canvas>
                        </div>
                    </div>
                    <p class="text-[11px] font-bold text-slate-700 mt-1.5">رمز الاستجابة السريع للتحقق الضريبي (ZATCA Phase 2)</p>
                </div>

            </div>
        </div>

        <!-- RECEIPT VIEW -->
        <div id="receiptView" class="hidden">
            <div id="receiptWrapper" class="page-border-wrapper receipt-border p-6 sm:p-8 bg-white relative min-h-[600px] text-right font-sans border-2 border-slate-800 rounded-2xl shadow-sm" dir="rtl">
                
                <div id="receiptStampContainer" class="draggable-stamp-container hidden" style="bottom: 30px; left: 120px;">
                    <div class="stamp-controls">
                        <div class="stamp-ctrl-btn stamp-btn-save" onclick="saveStampFromContainer('receiptStampContainer')" title="حفظ الختم في الفهرس المحلي"><i class="fa-solid fa-floppy-disk text-[10px]"></i></div>
                        <div class="stamp-ctrl-btn stamp-btn-rotate" title="تدوير الختم"><i class="fa-solid fa-rotate text-[10px]"></i></div>
                        <div class="stamp-ctrl-btn stamp-btn-delete" onclick="removeStamp('receiptStampContainer')" title="حذف الختم"><i class="fa-solid fa-xmark text-[10px]"></i></div>
                    </div>
                    <img id="receiptStampImg" src="" alt="ختم السند" class="w-32 h-32 object-contain opacity-95 pointer-events-none">
                    <div class="stamp-btn-resize" title="تكبير/تصغير الختم"></div>
                </div>

                <div class="flex justify-between items-start mb-4">
                    <div class="space-y-1">
                        <div class="w-20 h-20 mb-2 border border-slate-200 rounded-xl flex items-center justify-center p-1 bg-white shrink-0 overflow-hidden shadow-sm">
                            <img id="receiptLogo" src="https://via.placeholder.com/150?text=الشعار" alt="Logo" class="max-w-full max-h-full object-contain">
                        </div>
                        <h2 id="receiptSellerName" class="font-extrabold text-lg text-slate-900 editable-field" contenteditable="true">البائع</h2>
                        <div class="text-sm text-slate-700">
                            الرقم الضريبي : <span id="receiptSellerVat" class="font-mono editable-field" contenteditable="true">-</span>
                        </div>
                        <div class="text-sm text-slate-700">
                            العنوان: <span id="receiptSellerAddress" class="editable-field" contenteditable="true">-</span>
                        </div>
                    </div>

                    <div class="text-left space-y-1">
                        <h1 class="text-2xl font-black text-slate-900 mb-1">إيصال استلام</h1>
                        <div class="text-sm font-bold text-slate-800 flex items-center justify-end gap-1">
                            <span>رقم السند:</span>
                            <span id="receiptNumber" class="font-mono text-slate-900 border border-slate-300 rounded px-2 py-0.5 editable-field" contenteditable="true">2200</span>
                        </div>
                        <div class="text-sm text-slate-700 flex items-center justify-end gap-1">
                            <span>التاريخ:</span>
                            <span id="receiptDate" class="font-mono editable-field" contenteditable="true">12/08/2026</span>
                        </div>
                    </div>
                </div>

                <hr class="border-slate-800 border-t-2 my-4" />

                <div class="space-y-4 text-base leading-relaxed py-2 text-slate-800">
                    <div class="flex items-center">
                        <span class="font-bold w-24 shrink-0 text-slate-900">من:</span>
                        <span id="receiptFromClient" class="flex-1 border-b border-dashed border-slate-300 font-semibold editable-field" contenteditable="true">عميل نقدي</span>
                    </div>

                    <div class="flex items-center gap-8">
                        <div class="flex items-center flex-1">
                            <span class="font-bold w-24 shrink-0 text-slate-900">المبلغ:</span>
                            <span id="receiptAmount" class="font-bold font-mono text-slate-900 border-b border-dashed border-slate-300 editable-field" contenteditable="true">SAR 0.00</span>
                        </div>
                        <div class="flex items-center">
                            <span class="font-bold ml-2 text-slate-900">بواسطة:</span>
                            <span id="receiptByMethod" class="font-semibold border-b border-dashed border-slate-300 editable-field" contenteditable="true">نقداً</span>
                        </div>
                    </div>

                    <div class="flex items-center">
                        <span class="font-bold w-24 shrink-0 text-slate-900">فاتورة:</span>
                        <span id="receiptInvoiceNo" class="font-mono font-semibold border-b border-dashed border-slate-300 editable-field" contenteditable="true">INV-2355</span>
                    </div>

                    <div class="flex items-center">
                        <span class="font-bold w-24 shrink-0 text-slate-900">الخزينه:</span>
                        <span id="receiptTreasury" class="font-semibold border-b border-dashed border-slate-300 editable-field" contenteditable="true">Main</span>
                    </div>
                </div>

                <hr class="border-slate-400 my-6" />

                <div class="relative mt-4 flex justify-between items-start pb-6 min-h-[120px]">
                    <div></div>
                    <div class="text-left space-y-1 pl-4">
                        <div class="font-bold text-lg text-slate-900 flex items-center justify-end gap-1">
                            <span>:</span>
                            <span class="editable-field" contenteditable="true">التوقيع</span>
                        </div>
                    </div>
                </div>

            </div>
        </div>

    </div>

    <!-- Modals -->
    <!-- AI MODAL / شاشة الذكاء الاصطناعي والشات -->
    <div id="aiModal" class="no-print fixed inset-0 bg-black/60 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl w-full max-w-3xl p-5 shadow-2xl border border-slate-200 m-4 flex flex-col h-[85vh]">
            <div class="flex justify-between items-center mb-3 border-b pb-2 shrink-0">
                <h3 class="text-base font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-wand-magic-sparkles text-purple-600"></i> 
                    مساعد الأصناف والذكاء الاصطناعي (Gemini)
                </h3>
                <button onclick="closeAiModal()" class="text-slate-400 hover:text-slate-600 text-base font-bold"><i class="fa-solid fa-xmark"></i></button>
            </div>

            <!-- API KEY INPUT SECTION -->
            <div class="mb-3 p-2 bg-purple-50 rounded-xl border border-purple-100 flex items-center justify-between text-xs gap-2 shrink-0">
                <span class="text-purple-900 font-bold shrink-0"><i class="fa-solid fa-key"></i> Gemini API Key:</span>
                <input type="password" id="geminiApiKey" placeholder="أدخل مفتاح Gemini API هنا..." class="flex-1 bg-white border border-purple-200 rounded-lg px-2.5 py-1 text-xs outline-none focus:border-purple-500">
                <button onclick="saveApiKey()" class="bg-purple-600 hover:bg-purple-700 text-white font-bold px-3 py-1 rounded-lg transition text-xs shrink-0">حفظ المفتاح</button>
            </div>
            
            <!-- CHAT DISPLAY AREA -->
            <div id="chatBox" class="flex-1 overflow-y-auto space-y-3 p-3 bg-slate-50 rounded-xl border border-slate-200 mb-3 text-xs">
                <div class="bg-purple-100 text-purple-900 p-3 rounded-2xl max-w-[80%] ml-auto font-normal">
                    أهلاً بك يا صاحبي! أنا مساعد الأصناف الذكي. تقدر تطلب مني توليد أصناف وأسعار بطلب كتابي، أو ترفع صورة قائمة أسعار/فاتورة وأنا هستخرج الأصناف فوراً وأحفظهالك.
                </div>
            </div>

            <!-- AI GENERATED POOL OF PRODUCTS -->
            <div id="aiPoolContainer" class="max-h-40 overflow-y-auto mb-3 p-2 bg-white rounded-xl border border-slate-200 shrink-0">
                <div class="flex justify-between items-center mb-1 text-xs font-bold text-slate-700 border-b pb-1">
                    <span>قائمة الأصناف المحفوظة تلقائياً (<span id="aiPoolCount">0</span> صنف):</span>
                    <button onclick="clearAiPool()" class="text-rose-500 hover:text-rose-700 text-[11px]"><i class="fa-solid fa-trash-can"></i> تفريغ الكل</button>
                </div>
                <div id="aiPoolList" class="flex flex-wrap gap-1.5 text-[11px]">
                    <span class="text-slate-400 text-xs">لا توجد أصناف خازنة بعد. ابدأ المحادثة أو ارفع صورة!</span>
                </div>
            </div>

            <!-- CHAT INPUT CONTROLS -->
            <div class="flex items-center gap-2 shrink-0">
                <label class="bg-slate-100 hover:bg-slate-200 text-slate-700 p-2.5 rounded-xl border cursor-pointer transition flex items-center justify-center shrink-0" title="رفع صورة أصناف">
                    <i class="fa-solid fa-image text-purple-600 text-base"></i>
                    <input type="file" id="aiImageInput" accept="image/*" class="hidden" onchange="handleAiImageSelect(event)">
                </label>

                <input type="text" id="aiUserPrompt" onkeypress="if(event.key==='Enter') sendAiMessage()" placeholder="اكتب طلبك هنا... (مثلاً: توليد 15 صنف ساندوتشات ومشروبات)" class="flex-1 text-xs bg-slate-50 border border-slate-300 rounded-xl px-3 py-2.5 outline-none focus:border-purple-500 font-normal">

                <button onclick="sendAiMessage()" class="bg-purple-600 hover:bg-purple-700 text-white font-bold px-4 py-2.5 rounded-xl transition text-xs flex items-center gap-1.5 shrink-0">
                    <span>إرسال</span>
                    <i class="fa-solid fa-paper-plane"></i>
                </button>
            </div>
            <p id="selectedImgName" class="text-[10px] text-purple-600 mt-1 font-bold hidden"></p>
        </div>
    </div>

    <div id="partyModal" class="no-print fixed inset-0 bg-black/50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl w-full max-w-lg p-5 shadow-xl border border-slate-200 m-4">
            <div class="flex justify-between items-center mb-4 border-b pb-2">
                <h3 id="modalTitle" class="text-sm font-bold text-slate-800">فهرس الجهات</h3>
                <button onclick="closeModal()" class="text-slate-400 hover:text-slate-600 text-sm font-bold"><i class="fa-solid fa-xmark"></i></button>
            </div>
            
            <div class="mb-3">
                <input type="text" id="modalSearch" oninput="filterModalList()" placeholder="ابحث هنا..." class="w-full text-xs bg-slate-50 border border-slate-300 rounded-xl px-3 py-2 outline-none focus:border-blue-500">
            </div>

            <div id="modalListContainer" class="max-h-60 overflow-y-auto space-y-2 mb-4 text-xs"></div>

            <div class="flex justify-end">
                <button onclick="closeModal()" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-4 py-2 rounded-xl transition">إغلاق</button>
            </div>
        </div>
    </div>

    <div id="archiveModal" class="no-print fixed inset-0 bg-black/50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-2xl w-full max-w-2xl p-5 shadow-xl border border-slate-200 m-4">
            <div class="flex justify-between items-center mb-4 border-b pb-2">
                <h3 class="text-sm font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-box-archive text-indigo-600"></i> أرشيف الفواتير المحفوظة محلياً
                </h3>
                <button onclick="closeArchiveModal()" class="text-slate-400 hover:text-slate-600 text-sm font-bold"><i class="fa-solid fa-xmark"></i></button>
            </div>
            
            <div class="mb-3">
                <input type="text" id="archiveSearch" oninput="filterArchiveList()" placeholder="ابحث بـ (اسم العميل - رقم الفاتورة - التاريخ)..." class="w-full text-xs bg-slate-50 border border-slate-300 rounded-xl px-3 py-2 outline-none focus:border-indigo-500">
            </div>

            <div id="archiveListContainer" class="max-h-80 overflow-y-auto space-y-2 mb-4 text-xs"></div>

            <div class="flex justify-end">
                <button onclick="closeArchiveModal()" class="bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-4 py-2 rounded-xl transition">إغلاق</button>
            </div>
        </div>
    </div>

    <script>
        let localData = {
            saved_invoices: [],
            saved_sellers: [],
            saved_buyers: [],
            saved_stamps: [],
            saved_logos: [],
            ai_products_pool: [],
            gemini_api_key: ""
        };

        let selectedBase64Image = null;

        let lastKnownDate = "12/08/2026";
        let baseInvoiceSeq = 2355;
        let baseReceiptSeq = 2200;

        function saveLocalStorageData() {
            localStorage.setItem('local_invoice_app_backup', JSON.stringify(localData));
        }

        function loadLocalStorageData() {
            const local = localStorage.getItem('local_invoice_app_backup');
            if (local) {
                try { 
                    localData = JSON.parse(local); 
                    if (!localData.ai_products_pool) localData.ai_products_pool = [];
                } catch(e){}
            }
        }

        // --- ميزة الذكاء الاصطناعي والشات ---
        function openAiModal() {
            document.getElementById('aiModal').classList.remove('hidden');
            if (localData.gemini_api_key) {
                document.getElementById('geminiApiKey').value = localData.gemini_api_key;
            }
            renderAiPool();
        }

        function closeAiModal() {
            document.getElementById('aiModal').classList.add('hidden');
        }

        function saveApiKey() {
            const key = document.getElementById('geminiApiKey').value.trim();
            localData.gemini_api_key = key;
            saveLocalStorageData();
            alert("تم حفظ المفتاح محلياً بنجاح!");
        }

        function handleAiImageSelect(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(event) {
                    selectedBase64Image = event.target.result.split(',')[1];
                    document.getElementById('selectedImgName').innerText = "تم اختيار صورة: " + file.name;
                    document.getElementById('selectedImgName').classList.remove('hidden');
                };
                reader.readAsDataURL(file);
            }
        }

        function appendChatMessage(sender, text) {
            const chatBox = document.getElementById('chatBox');
            const msgDiv = document.createElement('div');
            
            if (sender === 'user') {
                msgDiv.className = "bg-blue-600 text-white p-3 rounded-2xl max-w-[80%] mr-auto font-normal";
            } else {
                msgDiv.className = "bg-purple-100 text-purple-900 p-3 rounded-2xl max-w-[80%] ml-auto font-normal";
            }
            msgDiv.innerText = text;
            chatBox.appendChild(msgDiv);
            chatBox.scrollTop = chatBox.scrollHeight;
        }

        async function sendAiMessage() {
            const apiKey = document.getElementById('geminiApiKey').value.trim();
            if (!apiKey) {
                alert("يرجى إدخال مفتاح Gemini API أولاً للحصول على خدمات الذكاء الاصطناعي!");
                return;
            }

            const promptInput = document.getElementById('aiUserPrompt');
            const userText = promptInput.value.trim();

            if (!userText && !selectedBase64Image) return;

            appendChatMessage('user', userText || "قام برفع صورة واستخراج الأصناف...");
            promptInput.value = '';
            showLoading("جاري الاتصال بالذكاء الاصطناعي وتحليل الطلب...");

            const systemInstruction = `أنت مساعد متخصص في استخراج وتوليد أصناف الفواتير. وظيفتك هي إرجاع الأغراض والأصناف بأسعارها فقط بتنسيق JSON حصري بدون أي نص إضافي أو مقدمات.التنسيق المطلوبة بالضبط كالتالي: [{"description": "اسم الصنف", "unitPrice": 25.0}]`;

            let contentsPayload = [];

            if (selectedBase64Image) {
                contentsPayload.push({
                    parts: [
                        { text: systemInstruction + " استخرج الأصناف والأسعار الموجودة في الصورة التالية:" },
                        { inline_data: { mime_type: "image/jpeg", data: selectedBase64Image } }
                    ]
                });
            } else {
                contentsPayload.push({
                    parts: [
                        { text: systemInstruction + " نفذ الطلب التالي وولد الأصناف المطلوبة مع أسعار منطقية بالريال/الجنيه: " + userText }
                    ]
                });
            }

            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: contentsPayload })
                });

                const data = await response.json();
                hideLoading();

                if (data.candidates && data.candidates[0].content.parts[0].text) {
                    let aiReply = data.candidates[0].content.parts[0].text;
                    
                    const jsonMatch = aiReply.match(/\[.*\]/s);
                    if (jsonMatch) {
                        try {
                            const parsedItems = JSON.parse(jsonMatch[0]);
                            if (Array.isArray(parsedItems) && parsedItems.length > 0) {
                                parsedItems.forEach(item => {
                                    if (item.description && item.unitPrice) {
                                        localData.ai_products_pool.push({
                                            description: item.description,
                                            unitPrice: parseFloat(item.unitPrice) || 0
                                        });
                                    }
                                });
                                saveLocalStorageData();
                                renderAiPool();
                                appendChatMessage('ai', `تم استخراج وتوليد (${parsedItems.length}) صنف بنجاح وإضافتهم لقائمة الأصناف المتاحة!`);
                            }
                        } catch(e) {
                            appendChatMessage('ai', "تم الحصول على الرد لكن تعذر تنسيق الأصناف تلقائياً. " + aiReply);
                        }
                    } else {
                        appendChatMessage('ai', aiReply);
                    }
                } else {
                    appendChatMessage('ai', "لم يتم استلام رد صالح من الذكاء الاصطناعي.");
                }

            } catch (error) {
                hideLoading();
                console.error(error);
                appendChatMessage('ai', "حدث خطأ أثناء الاتصال بالذكاء الاصطناعي. تأكد من صحة الـ API Key والاتصال بالنت.");
            }

            selectedBase64Image = null;
            document.getElementById('selectedImgName').classList.add('hidden');
            document.getElementById('aiImageInput').value = '';
        }

        function renderAiPool() {
            const container = document.getElementById('aiPoolList');
            const countSpan = document.getElementById('aiPoolCount');
            container.innerHTML = '';

            let pool = localData.ai_products_pool || [];
            countSpan.innerText = pool.length;

            if (pool.length === 0) {
                container.innerHTML = '<span class="text-slate-400 text-xs">لا توجد أصناف مخزنة بعد. ابدأ المحادثة أو ارفع صورة!</span>';
                return;
            }

            pool.forEach((item, idx) => {
                const tag = document.createElement('div');
                tag.className = "bg-slate-100 hover:bg-purple-50 text-slate-800 border border-slate-300 rounded-lg px-2 py-1 flex items-center gap-1.5 font-bold";
                tag.innerHTML = `
                    <span>${item.description}</span>
                    <span class="text-emerald-600 font-mono">(${item.unitPrice} ر.س)</span>
                    <button onclick="removeAiPoolItem(${idx})" class="text-rose-500 hover:text-rose-700 ml-1" title="حذف هذا الصنف"><i class="fa-solid fa-xmark"></i></button>
                `;
                container.appendChild(tag);
            });
        }

        function removeAiPoolItem(idx) {
            localData.ai_products_pool.splice(idx, 1);
            saveLocalStorageData();
            renderAiPool();
        }

        function clearAiPool() {
            if (confirm("هل أنت متأكد من مسح جميع الأصناف المحفوظة بالذكاء الاصطناعي؟")) {
                localData.ai_products_pool = [];
                saveLocalStorageData();
                renderAiPool();
            }
        }

        // --- ملء جدول الفاتورة عشوائياً بالأصناف المولدة ---
        function fillInvoiceWithAiPool() {
            let pool = localData.ai_products_pool || [];
            if (pool.length === 0) {
                alert("قائمة الأصناف المحفوظة فارغة! اضغط على زر 'مساعد الأصناف والذكاء الاصطناعي' وقم بتوليد أصناف أولاً.");
                return;
            }

            const currentRowCount = items.length;

            let shuffledPool = [...pool].sort(() => 0.5 - Math.random());

            for (let i = 0; i < currentRowCount; i++) {
                const pickedItem = shuffledPool[i % shuffledPool.length];
                items[i].description = pickedItem.description;
                items[i].unitPrice = pickedItem.unitPrice;
            }

            renderTable();
            alert(`تم ملء (${currentRowCount}) خانة بالأصناف المتاحة عشوائياً بنجاح!`);
        }

        function updateSequenceRealtime() {
            const currentDateStr = document.getElementById('stdInvDate').innerText.trim();
            if (!currentDateStr || currentDateStr === lastKnownDate) return;

            const partsNew = currentDateStr.split('/');
            const partsOld = lastKnownDate.split('/');

            if (partsNew.length === 3 && partsOld.length === 3) {
                const dateNew = new Date(partsNew[2], partsNew[1] - 1, partsNew[0]);
                const dateOld = new Date(partsOld[2], partsOld[1] - 1, partsOld[0]);

                if (!isNaN(dateNew.getTime()) && !isNaN(dateOld.getTime())) {
                    const timeDiff = dateNew.getTime() - dateOld.getTime();
                    const daysDiff = Math.round(timeDiff / (1000 * 3600 * 24));

                    let invInc = 0;
                    let recInc = 0;

                    if (dateNew.getMonth() === dateOld.getMonth() && dateNew.getFullYear() === dateOld.getFullYear()) {
                        if (daysDiff !== 0) {
                            invInc = daysDiff * (Math.floor(Math.random() * 10) + 18);
                        }
                    } else {
                        const monthDiff = (dateNew.getFullYear() - dateOld.getFullYear()) * 12 + (dateNew.getMonth() - dateOld.getMonth());
                        invInc = monthDiff * (Math.floor(Math.random() * 300) + 400);
                    }

                    if (invInc !== 0) {
                        recInc = Math.floor(invInc * 0.85);
                        baseInvoiceSeq = Math.max(1000, baseInvoiceSeq + invInc);
                        baseReceiptSeq = Math.max(800, baseReceiptSeq + recInc);

                        const formattedInv = "INV-" + baseInvoiceSeq;
                        const formattedRec = String(baseReceiptSeq).padStart(6, '0');

                        document.getElementById('stdInvNum').innerText = formattedInv;
                        document.getElementById('tpl4InvNum').innerText = formattedInv;
                        document.getElementById('receiptInvoiceNo').innerText = formattedInv;
                        document.getElementById('receiptNumber').innerText = formattedRec;

                        lastKnownDate = currentDateStr;
                        renderTable();
                    }
                }
            }
        }

        function openNewInvoiceTab() {
            updateSequenceRealtime();

            const currentInvVal = document.getElementById('stdInvNum').innerText.trim();
            const currentRecVal = document.getElementById('receiptNumber').innerText.trim();
            const currentDateVal = document.getElementById('stdInvDate').innerText.trim();

            const urlParams = new URLSearchParams();
            urlParams.set('inv', currentInvVal);
            urlParams.set('rec', currentRecVal);
            urlParams.set('date', currentDateVal);

            const newTabUrl = window.location.pathname + '?' + urlParams.toString();
            window.open(newTabUrl, '_blank');
        }

        function checkUrlParamsAndApply() {
            const urlParams = new URLSearchParams(window.location.search);
            if (urlParams.has('inv')) {
                const invVal = urlParams.get('inv');
                document.getElementById('stdInvNum').innerText = invVal;
                document.getElementById('tpl4InvNum').innerText = invVal;
                document.getElementById('receiptInvoiceNo').innerText = invVal;
                baseInvoiceSeq = parseInt(invVal.replace(/\D/g, '')) || 2355;
            }
            if (urlParams.has('rec')) {
                const recVal = urlParams.get('rec');
                document.getElementById('receiptNumber').innerText = recVal;
                baseReceiptSeq = parseInt(recVal.replace(/\D/g, '')) || 2200;
            }
            if (urlParams.has('date')) {
                const dateVal = urlParams.get('date');
                document.getElementById('stdInvDate').innerText = dateVal;
                document.getElementById('tpl4InvDate').innerText = dateVal;
                document.getElementById('receiptDate').innerText = dateVal;
                lastKnownDate = dateVal;
            }
        }

        function convertFileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result);
                reader.onerror = error => reject(error);
            });
        }

        function showLoading(msg) {
            document.getElementById('loadingText').innerText = msg || "جاري التحميل...";
            document.getElementById('loadingOverlay').classList.remove('hidden');
        }

        function hideLoading() {
            document.getElementById('loadingOverlay').classList.add('hidden');
        }

        let items = [{ description: "", qty: 1, unitPrice: 0 }];
        let currentTemplate = 1;
        let currentView = 'invoice';

        let phase2Data = {
            hash: "NWZlYTY4M2E4...[Base64 encoded hash]",
            sig: "MEQCIEC...[ECDSA Signature Bytes/Base64]",
            key: "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE...[Public Key Bytes]"
        };

        function syncInvoiceToReceipt() {
            const sellerName = document.getElementById('sellerName').innerText.trim();
            const sellerVat = document.getElementById('sellerVat').innerText.trim();
            const sellerAddress = document.getElementById('sellerAddress').innerText.trim();
            const logoSrc = document.getElementById('companyLogo').src;

            const buyerName = document.getElementById('buyerName').innerText.trim();
            const invNum = document.getElementById('stdInvNum').innerText.trim();
            const invDate = document.getElementById('stdInvDate').innerText.trim();
            const grandTotal = document.getElementById('grandTotalDisplay').innerText.trim();
            const payMethod = document.getElementById('paymentMethod').innerText.trim();

            if (sellerName) document.getElementById('receiptSellerName').innerText = sellerName;
            if (sellerVat) document.getElementById('receiptSellerVat').innerText = sellerVat;
            if (sellerAddress) document.getElementById('receiptSellerAddress').innerText = sellerAddress;
            if (logoSrc) document.getElementById('receiptLogo').src = logoSrc;

            if (buyerName) {
                document.getElementById('receiptFromClient').innerText = buyerName;
            }
            if (invNum) document.getElementById('receiptInvoiceNo').innerText = invNum;
            if (invDate) document.getElementById('receiptDate').innerText = invDate;
            if (grandTotal) document.getElementById('receiptAmount').innerText = 'SAR ' + grandTotal.replace(' ر.س', '');
            if (payMethod) document.getElementById('receiptByMethod').innerText = payMethod;
        }

        function switchView(view) {
            currentView = view;
            const invoiceView = document.getElementById('invoiceView');
            const receiptView = document.getElementById('receiptView');
            const btnInv = document.getElementById('btnViewInvoice');
            const btnRec = document.getElementById('btnViewReceipt');
            const templateControls = document.getElementById('templateControls');

            if (view === 'invoice') {
                invoiceView.classList.remove('hidden');
                receiptView.classList.add('hidden');
                templateControls.classList.remove('hidden');
                btnInv.className = "bg-slate-800 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5";
                btnRec.className = "bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5";
            } else {
                syncInvoiceToReceipt();
                invoiceView.classList.add('hidden');
                receiptView.classList.remove('hidden');
                templateControls.classList.add('hidden');
                btnRec.className = "bg-amber-600 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5";
                btnInv.className = "bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5";
            }
        }

        function removeStamp(containerId) {
            const container = document.getElementById(containerId);
            container.classList.add('hidden');
            const img = container.querySelector('img');
            if(img) img.src = '';
        }

        async function saveStampFromContainer(containerId) {
            const img = document.getElementById(containerId.replace('Container', 'Img'));
            if (!img || !img.src || img.src.includes('via.placeholder.com')) {
                alert('لا يوجد ختم صالح للحفظ!');
                return;
            }
            const stampName = prompt("أدخل اسماً لهذا الختم لحفظه في الفهرس المحلي:", "ختم جديد");
            if (!stampName) return;

            if(!localData.saved_stamps) localData.saved_stamps = [];
            localData.saved_stamps.push({ name: stampName, src: img.src });
            saveLocalStorageData();
            alert('تم حفظ الختم في الفهرس المحلي بنجاح!');
        }

        function setupAdvancedStamp(containerId) {
            const container = document.getElementById(containerId);
            const parentWrapper = container.offsetParent || document.body;
            const img = container.querySelector('img');
            const rotateBtn = container.querySelector('.stamp-btn-rotate');
            const resizeBtn = container.querySelector('.stamp-btn-resize');

            let currentRotation = 0;

            container.addEventListener('click', function(e) {
                document.querySelectorAll('.draggable-stamp-container').forEach(el => el.classList.remove('active-stamp'));
                container.classList.add('active-stamp');
                e.stopPropagation();
            });

            document.addEventListener('click', function() {
                container.classList.remove('active-stamp');
            });

            function getPos(e) {
                if (e.touches && e.touches.length > 0) {
                    return { clientX: e.touches[0].clientX, clientY: e.touches[0].clientY };
                }
                return { clientX: e.clientX, clientY: e.clientY };
            }

            function startDrag(e) {
                if (e.target.classList.contains('stamp-ctrl-btn') || e.target.classList.contains('stamp-btn-resize') || e.target.closest('.stamp-controls')) return;

                let pos = getPos(e);
                let rect = container.getBoundingClientRect();
                let parentRect = parentWrapper.getBoundingClientRect();

                let offsetX = pos.clientX - rect.left;
                let offsetY = pos.clientY - rect.top;

                function onMove(e) {
                    let movePos = getPos(e);
                    let left = movePos.clientX - parentRect.left - offsetX;
                    let top = movePos.clientY - parentRect.top - offsetY;

                    container.style.left = left + 'px';
                    container.style.top = top + 'px';
                    container.style.bottom = 'auto';
                }

                function onEnd() {
                    document.removeEventListener('mousemove', onMove);
                    document.removeEventListener('mouseup', onEnd);
                    document.removeEventListener('touchmove', onMove);
                    document.removeEventListener('touchend', onEnd);
                }

                document.addEventListener('mousemove', onMove);
                document.addEventListener('mouseup', onEnd);
                document.addEventListener('touchmove', onMove, { passive: false });
                document.addEventListener('touchend', onEnd);
            }

            container.addEventListener('mousedown', startDrag);
            container.addEventListener('touchstart', startDrag, { passive: false });

            if (rotateBtn) {
                function startRotate(e) {
                    e.stopPropagation();
                    const rect = container.getBoundingClientRect();
                    const centerX = rect.left + rect.width / 2;
                    const centerY = rect.top + rect.height / 2;

                    function onRotateMove(e) {
                        let pos = getPos(e);
                        const radians = Math.atan2(pos.clientY - centerY, pos.clientX - centerX);
                        let degrees = radians * (180 / Math.PI) + 90;
                        currentRotation = degrees;
                        container.style.transform = `rotate(${degrees}deg)`;
                    }

                    function onRotateUp() {
                        document.removeEventListener('mousemove', onRotateMove);
                        document.removeEventListener('mouseup', onRotateUp);
                        document.removeEventListener('touchmove', onRotateMove);
                        document.removeEventListener('touchend', onRotateUp);
                    }

                    document.addEventListener('mousemove', onRotateMove);
                    document.addEventListener('mouseup', onRotateUp);
                    document.addEventListener('touchmove', onRotateMove, { passive: false });
                    document.addEventListener('touchend', onRotateUp);
                }

                rotateBtn.addEventListener('mousedown', startRotate);
                rotateBtn.addEventListener('touchstart', startRotate, { passive: false });
            }

            if (resizeBtn) {
                function startResize(e) {
                    e.stopPropagation();
                    let startPos = getPos(e);
                    let startWidth = img.offsetWidth;

                    function onResizeMove(e) {
                        let pos = getPos(e);
                        let newWidth = startWidth + (pos.clientX - startPos.clientX);
                        if (newWidth > 40 && newWidth < 300) {
                            img.style.width = newWidth + 'px';
                            img.style.height = newWidth + 'px';
                        }
                    }

                    function onResizeUp() {
                        document.removeEventListener('mousemove', onResizeMove);
                        document.removeEventListener('mouseup', onResizeUp);
                        document.removeEventListener('touchmove', onResizeMove);
                        document.removeEventListener('touchend', onResizeUp);
                    }

                    document.addEventListener('mousemove', onResizeMove);
                    document.addEventListener('mouseup', onResizeUp);
                    document.addEventListener('touchmove', onResizeMove, { passive: false });
                    document.addEventListener('touchend', onResizeUp);
                }

                resizeBtn.addEventListener('mousedown', startResize);
                resizeBtn.addEventListener('touchstart', startResize, { passive: false });
            }
        }

        document.getElementById('receiptStampUpload').addEventListener('change', async function(e) {
            const file = e.target.files[0];
            if (file) {
                const base64Url = await convertFileToBase64(file);
                const container = document.getElementById('receiptStampContainer');
                document.getElementById('receiptStampImg').src = base64Url;
                container.classList.remove('hidden');
                setupAdvancedStamp('receiptStampContainer');
            }
        });

        document.getElementById('invoiceStampUpload').addEventListener('change', async function(e) {
            const file = e.target.files[0];
            if (file) {
                const base64Url = await convertFileToBase64(file);
                const container = document.getElementById('invoiceStampContainer');
                document.getElementById('invoiceStampImg').src = base64Url;
                container.classList.remove('hidden');
                setupAdvancedStamp('invoiceStampContainer');
            }
        });

        function addCustomField(containerId) {
            const container = document.getElementById(containerId);
            const p = document.createElement('p');
            p.className = "flex justify-between items-center relative group w-full";
            p.innerHTML = `
                <button onclick="this.closest('p').remove()" class="no-print text-red-400 hover:text-red-600 opacity-0 group-hover:opacity-100 transition text-[10px] absolute -right-4" title="حذف الحقل"><i class="fa-solid fa-xmark"></i></button>
                <span class="text-slate-500 editable-field font-normal" contenteditable="true">اسم الحقل:</span> 
                <span class="text-slate-800 editable-field font-normal" contenteditable="true">البيانات</span>
            `;
            container.appendChild(p);
        }

        document.getElementById('stdInvDate').addEventListener('blur', updateSequenceRealtime);
        document.getElementById('stdInvDate').addEventListener('input', function() {
            document.getElementById('tpl4InvDate').innerText = this.innerText;
            syncInvoiceToReceipt();
        });

        document.getElementById('tpl4InvDate').addEventListener('blur', updateSequenceRealtime);
        document.getElementById('tpl4InvDate').addEventListener('input', function() {
            document.getElementById('stdInvDate').innerText = this.innerText;
            syncInvoiceToReceipt();
        });

        document.getElementById('stdInvNum').addEventListener('input', function() {
            document.getElementById('tpl4InvNum').innerText = this.innerText;
            syncInvoiceToReceipt();
        });
        document.getElementById('tpl4InvNum').addEventListener('input', function() {
            document.getElementById('stdInvNum').innerText = this.innerText;
            syncInvoiceToReceipt();
        });

        function switchTemplate(tplNum) {
            currentTemplate = tplNum;
            const wrapper = document.getElementById('invoiceWrapper');
            const btn1 = document.getElementById('btnTpl1');
            const btn2 = document.getElementById('btnTpl2');
            const btn3 = document.getElementById('btnTpl3');
            const btn4 = document.getElementById('btnTpl4');
            const qrContainer = document.getElementById('qrcodeContainer');

            btn1.className = "bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5";
            btn2.className = "bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5";
            btn3.className = "bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5";
            btn4.className = "bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-lg transition flex items-center gap-1.5";

            wrapper.classList.remove('tpl-1', 'tpl-2', 'tpl-3', 'tpl-4');

            if (tplNum === 1) {
                btn1.className = "bg-slate-800 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5";
                wrapper.classList.add('tpl-1');

                document.getElementById('tpl1QrSlot').appendChild(qrContainer);
                document.getElementById('tpl1QrSection').style.display = 'flex';
                document.getElementById('tpl2QrSection').style.display = 'none';
                document.getElementById('tpl4TitleBlock').classList.add('hidden');
                document.getElementById('tpl4MetaCenter').classList.add('hidden');
                document.getElementById('tpl4QrSlot').classList.add('hidden');
                document.getElementById('tpl4QrSlot').classList.remove('flex');
                document.getElementById('stdInvNumRow').style.display = 'flex';
                document.getElementById('stdInvDateRow').style.display = 'flex';
            } else if (tplNum === 2) {
                btn2.className = "bg-teal-700 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5";
                wrapper.classList.add('tpl-2');

                document.getElementById('tpl2QrSlot').appendChild(qrContainer);
                document.getElementById('tpl1QrSection').style.display = 'none';
                document.getElementById('tpl2QrSection').style.display = 'flex';
                document.getElementById('tpl4TitleBlock').classList.add('hidden');
                document.getElementById('tpl4MetaCenter').classList.add('hidden');
                document.getElementById('tpl4QrSlot').classList.add('hidden');
                document.getElementById('tpl4QrSlot').classList.remove('flex');
                document.getElementById('stdInvNumRow').style.display = 'flex';
                document.getElementById('stdInvDateRow').style.display = 'flex';
            } else if (tplNum === 3) {
                btn3.className = "bg-[#0096d6] text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5";
                wrapper.classList.add('tpl-3');

                document.getElementById('tpl2QrSlot').appendChild(qrContainer);
                document.getElementById('tpl1QrSection').style.display = 'none';
                document.getElementById('tpl2QrSection').style.display = 'flex';
                document.getElementById('tpl4TitleBlock').classList.add('hidden');
                document.getElementById('tpl4MetaCenter').classList.add('hidden');
                document.getElementById('tpl4QrSlot').classList.add('hidden');
                document.getElementById('tpl4QrSlot').classList.remove('flex');
                document.getElementById('stdInvNumRow').style.display = 'flex';
                document.getElementById('stdInvDateRow').style.display = 'flex';
            } else if (tplNum === 4) {
                btn4.className = "bg-sky-600 text-white text-xs font-bold px-3 py-1.5 rounded-lg shadow transition flex items-center gap-1.5";
                wrapper.classList.add('tpl-4');

                document.getElementById('tpl4QrSlot').insertBefore(qrContainer, document.getElementById('tpl4QrSlot').firstChild);
                document.getElementById('tpl4QrSlot').classList.remove('hidden');
                document.getElementById('tpl4QrSlot').classList.add('flex');
                document.getElementById('tpl4TitleBlock').classList.remove('hidden');
                document.getElementById('tpl4MetaCenter').classList.remove('hidden');
                
                document.getElementById('stdInvNumRow').style.display = 'none';
                document.getElementById('stdInvDateRow').style.display = 'none';
                
                document.getElementById('tpl1QrSection').style.display = 'none';
                document.getElementById('tpl2QrSection').style.display = 'none';
            }
        }

        function renderTable() {
            const tbody = document.getElementById('itemsBody');
            tbody.innerHTML = '';

            let subTotal = 0;
            let vatTotal = 0;

            items.forEach((item, index) => {
                const rowTotalExcl = item.qty * item.unitPrice;
                const vatAmount = rowTotalExcl * 0.15;
                const totalIncl = rowTotalExcl + vatAmount;

                subTotal += rowTotalExcl;
                vatTotal += vatAmount;

                const row = document.createElement('tr');
                row.className = "hover:bg-slate-50 transition";
                row.innerHTML = `
                    <td class="p-2">
                        <input type="text" value="${item.description}" placeholder="اسم الصنف أو الخدمة" onchange="updateItem(${index}, 'description', this.value)" class="w-full bg-transparent border-b border-transparent hover:border-slate-300 focus:border-blue-500 focus:bg-white outline-none p-0.5 rounded font-normal text-slate-800">
                    </td>
                    <td class="p-2 text-center">
                        <input type="number" value="${item.qty}" min="1" onchange="updateItem(${index}, 'qty', this.value)" class="w-full min-w-[40px] text-center bg-transparent border-b border-transparent hover:border-slate-300 focus:border-blue-500 focus:bg-white outline-none p-0.5 rounded font-mono font-normal text-slate-800">
                    </td>
                    <td class="p-2 text-center">
                        <input type="number" step="0.01" value="${item.unitPrice.toFixed(2)}" onchange="updateItem(${index}, 'unitPrice', this.value)" class="w-full min-w-[50px] text-center bg-transparent border-b border-transparent hover:border-slate-300 focus:border-blue-500 focus:bg-white outline-none p-0.5 rounded font-mono font-normal text-slate-800">
                    </td>
                    <td class="p-2 text-center font-mono font-normal text-slate-800">${rowTotalExcl.toFixed(2)}</td>
                    <td class="p-2 text-center font-mono font-normal text-slate-800">${vatAmount.toFixed(2)}</td>
                    <td class="p-2 text-center font-mono font-normal text-slate-800">${totalIncl.toFixed(2)}</td>
                    <td class="p-1 text-center no-print">
                        <button onclick="removeItem(${index})" class="text-rose-500 hover:text-rose-700 p-1"><i class="fa-solid fa-trash-can"></i></button>
                    </td>
                `;
                tbody.appendChild(row);
            });

            const grandTotal = subTotal + vatTotal;
            const formattedGrandTotal = grandTotal.toLocaleString('en-US', {minimumFractionDigits: 2}) + ' ر.س';

            document.getElementById('subTotalDisplay').innerText = subTotal.toLocaleString('en-US', {minimumFractionDigits: 2}) + ' ر.س';
            document.getElementById('vatTotalDisplay').innerText = vatTotal.toLocaleString('en-US', {minimumFractionDigits: 2}) + ' ر.س';
            document.getElementById('grandTotalDisplay').innerText = formattedGrandTotal;

            document.getElementById('paidDisplay').innerText = formattedGrandTotal;
            document.getElementById('dueDisplay').innerText = '0.00 ر.س';

            generateQRCode(grandTotal, vatTotal);
            syncInvoiceToReceipt();
        }

        function updateItem(index, field, value) {
            if (field === 'qty') items[index].qty = parseFloat(value) || 0;
            else if (field === 'unitPrice') items[index].unitPrice = parseFloat(value) || 0;
            else items[index][field] = value;
            renderTable();
        }

        function addNewRow() {
            items.push({ description: "", qty: 1, unitPrice: 0 });
            renderTable();
        }

        function removeItem(index) {
            if (items.length > 1) {
                items.splice(index, 1);
                renderTable();
            }
        }

        function createTLV(tagNum, valueStr) {
            const encoder = new TextEncoder();
            const valueBytes = encoder.encode(valueStr);
            const tag = new Uint8Array([tagNum]);
            const length = new Uint8Array([valueBytes.length]);
            
            const tlv = new Uint8Array(tag.length + length.length + valueBytes.length);
            tlv.set(tag, 0);
            tlv.set(length, tag.length);
            tlv.set(valueBytes, tag.length + length.length);
            return tlv;
        }

        function concatUint8Arrays(arrays) {
            let totalLength = arrays.reduce((acc, curr) => acc + curr.length, 0);
            let result = new Uint8Array(totalLength);
            let offset = 0;
            for (let arr of arrays) {
                result.set(arr, offset);
                offset += arr.length;
            }
            return result;
        }

        function uint8ToBase64(bytes) {
            let binary = '';
            let len = bytes.byteLength;
            for (let i = 0; i < len; i++) {
                binary += String.fromCharCode(bytes[i]);
            }
            return window.btoa(binary);
        }

        function getRandomTimestamp(baseDateStr) {
            let dateObj = new Date();
            
            if (baseDateStr) {
                const parts = baseDateStr.split('/');
                if (parts.length === 3) {
                    dateObj = new Date(`${parts[2]}-${parts[1]}-${parts[0]}`);
                }
            }
            if (isNaN(dateObj.getTime())) {
                dateObj = new Date();
            }

            const randomHour = Math.floor(Math.random() * 3) + 9;
            const randomMinute = Math.floor(Math.random() * 60);
            const randomSecond = Math.floor(Math.random() * 60);

            dateObj.setHours(randomHour, randomMinute, randomSecond);
            return dateObj.toISOString();
        }

        function generateQRCode(totalAmount, vatAmount) {
            const seller = document.getElementById('sellerName').innerText.trim() || " ";
            const vat = document.getElementById('sellerVat').innerText.trim() || "000000000000000";
            const dateStr = document.getElementById('stdInvDate').innerText.trim();
            const timestamp = getRandomTimestamp(dateStr);

            const fullTLV = concatUint8Arrays([
                createTLV(1, seller),
                createTLV(2, vat),
                createTLV(3, timestamp),
                createTLV(4, totalAmount.toFixed(2)),
                createTLV(5, vatAmount.toFixed(2)),
                createTLV(6, phase2Data.hash),
                createTLV(7, phase2Data.sig),
                createTLV(8, phase2Data.key)
            ]);

            const qrPayload = uint8ToBase64(fullTLV);

            var qr = qrcode(0, 'M');
            qr.addData(qrPayload);
            qr.make();

            var moduleCount = qr.getModuleCount();
            var boxSize = 5;
            var quietZoneModules = 2;    
            var innerWhitespacePx = 3;  
            var outerBorderPx = 1;       

            var rawQrSize = moduleCount * boxSize;
            var quietZoneSize = quietZoneModules * boxSize;
            var totalQrCoreSize = rawQrSize + (quietZoneSize * 2);
            var canvasSize = totalQrCoreSize + (innerWhitespacePx * 2) + (outerBorderPx * 2);

            var canvas = document.getElementById('qrCanvas');
            canvas.width = canvasSize;
            canvas.height = canvasSize;
            var ctx = canvas.getContext('2d');

            ctx.fillStyle = '#D1D5DB';
            ctx.fillRect(0, 0, canvasSize, canvasSize);

            ctx.fillStyle = '#FFFFFF';
            ctx.fillRect(
                outerBorderPx, 
                outerBorderPx, 
                canvasSize - (outerBorderPx * 2), 
                canvasSize - (outerBorderPx * 2)
            );

            var startX = outerBorderPx + innerWhitespacePx + quietZoneSize;
            var startY = outerBorderPx + innerWhitespacePx + quietZoneSize;

            ctx.fillStyle = '#000000';
            for (var r = 0; r < moduleCount; r++) {
                for (var c = 0; c < moduleCount; c++) {
                    if (qr.isDark(r, c)) {
                        ctx.fillRect(
                            startX + (c * boxSize), 
                            startY + (r * boxSize), 
                            boxSize, 
                            boxSize
                        );
                    }
                }
            }
        }

        function triggerGenerateQR() {
            let subTotal = 0;
            let vatTotal = 0;

            items.forEach(item => {
                const rowTotalExcl = item.qty * item.unitPrice;
                const vatAmount = rowTotalExcl * 0.15;
                subTotal += rowTotalExcl;
                vatTotal += vatAmount;
            });

            const grandTotal = subTotal + vatTotal;
            generateQRCode(grandTotal, vatTotal);
            alert("تم إعادة توليد الـ QR بنجاح!");
        }

        document.getElementById('logoUpload').addEventListener('change', async function(e) {
            const file = e.target.files[0];
            if (file) {
                const base64Url = await convertFileToBase64(file);
                document.getElementById('companyLogo').src = base64Url;
                syncInvoiceToReceipt();
            }
        });

        async function saveLogo() {
            const logoSrc = document.getElementById('companyLogo').src;
            if (logoSrc.includes('via.placeholder.com') && !confirm('هذا الشعار افتراضي، هل تريد حفظه؟')) {
                return;
            }
            const logoName = prompt("أدخل اسماً لهذا الشعار لحفظه في الفهرس المحلي:", "شعار جديد");
            if (!logoName) return;

            if(!localData.saved_logos) localData.saved_logos = [];
            localData.saved_logos.push({ name: logoName, src: logoSrc });
            saveLocalStorageData();
            alert('تم حفظ الشعار في الفهرس المحلي بنجاح!');
        }

        function selectLogo(src) {
            document.getElementById('companyLogo').src = src;
            syncInvoiceToReceipt();
            closeModal();
        }

        async function deleteLogo(index) {
            if(confirm('هل أنت متأكد من حذف هذا الشعار من الفهرس المحلي؟')) {
                localData.saved_logos.splice(index, 1);
                saveLocalStorageData();
                renderModalList(document.getElementById('modalSearch').value.toLowerCase());
            }
        }

        function selectStamp(src) {
            if (currentView === 'invoice') {
                const container = document.getElementById('invoiceStampContainer');
                document.getElementById('invoiceStampImg').src = src;
                container.classList.remove('hidden');
                setupAdvancedStamp('invoiceStampContainer');
            } else {
                const container = document.getElementById('receiptStampContainer');
                document.getElementById('receiptStampImg').src = src;
                container.classList.remove('hidden');
                setupAdvancedStamp('receiptStampContainer');
            }
            closeModal();
        }

        async function deleteStamp(index) {
            if(confirm('هل أنت متأكد من حذف هذا الختم من الفهرس المحلي؟')) {
                localData.saved_stamps.splice(index, 1);
                saveLocalStorageData();
                renderModalList(document.getElementById('modalSearch').value.toLowerCase());
            }
        }

        async function deleteParty(type, index) {
            const partyTypeLabel = (type === 'seller') ? 'البائع' : 'العميل';
            if (confirm(`هل أنت متأكد من حذف هذا ${partyTypeLabel} من الفهرس المحلي؟`)) {
                const key = type === 'seller' ? 'saved_sellers' : 'saved_buyers';
                localData[key].splice(index, 1);
                saveLocalStorageData();
                renderModalList(document.getElementById('modalSearch').value.toLowerCase());
            }
        }

        async function saveCurrentParty(type) {
            const name = document.getElementById(type + 'Name').innerText.trim();
            const vat = document.getElementById(type + 'Vat').innerText.trim();
            const cr = document.getElementById(type + 'Cr').innerText.trim();
            const address = document.getElementById(type + 'Address').innerText.trim();

            if (!name) {
                alert('يرجى إدخال اسم صحيح قبل الحفظ');
                return;
            }

            const key = type === 'seller' ? 'saved_sellers' : 'saved_buyers';
            if (!localData[key]) localData[key] = [];

            const existingIndex = localData[key].findIndex(item => item.name === name || item.vat === vat);
            const partyData = { name, vat, cr, address };

            if (existingIndex > -1) {
                localData[key][existingIndex] = partyData;
            } else {
                localData[key].push(partyData);
            }

            saveLocalStorageData();
            alert('تم حفظ بيانات ' + (type === 'seller' ? 'البائع' : 'العميل') + ' محلياً بنجاح!');
        }

        let currentModalType = '';

        function openModal(type) {
            currentModalType = type;
            
            if (type === 'seller') document.getElementById('modalTitle').innerText = 'فهرس البائعين (محلي)';
            else if (type === 'buyer') document.getElementById('modalTitle').innerText = 'فهرس العملاء (محلي)';
            else if (type === 'logo') document.getElementById('modalTitle').innerText = 'فهرس الشعارات (محلي)';
            else if (type === 'stamp') document.getElementById('modalTitle').innerText = 'فهرس الأختام (محلي)';

            document.getElementById('modalSearch').value = '';
            document.getElementById('modalSearch').placeholder = (type === 'logo' || type === 'stamp') ? 'ابحث بالاسم...' : 'ابحث بالاسم أو الرقم الضريبي...';
            
            renderModalList('');
            document.getElementById('partyModal').classList.remove('hidden');
        }

        function closeModal() {
            document.getElementById('partyModal').classList.add('hidden');
        }

        function filterModalList() {
            const query = document.getElementById('modalSearch').value.toLowerCase();
            renderModalList(query);
        }

        function renderModalList(searchQuery) {
            const container = document.getElementById('modalListContainer');
            container.innerHTML = '';

            if (currentModalType === 'stamp') {
                let list = localData.saved_stamps || [];
                const filtered = list.filter(item => item.name.toLowerCase().includes(searchQuery));

                if (filtered.length === 0) {
                    container.innerHTML = '<p class="text-center text-slate-400 py-4">لا توجد أختام محفوظة محلياً</p>';
                    return;
                }

                filtered.forEach((item, index) => {
                    const div = document.createElement('div');
                    div.className = "bg-slate-50 hover:bg-emerald-50/50 p-2 rounded-xl border border-slate-200 flex justify-between items-center transition";
                    div.innerHTML = `
                        <div class="flex items-center gap-3 overflow-hidden">
                            <div class="w-12 h-12 bg-white border border-slate-200 rounded flex items-center justify-center shrink-0">
                                <img src="${item.src}" class="max-h-full max-w-full object-contain">
                            </div>
                            <p class="font-bold text-slate-900 truncate">${item.name}</p>
                        </div>
                        <div class="flex items-center gap-1 shrink-0">
                            <button onclick="deleteStamp(${index})" class="text-red-500 hover:text-red-700 p-1.5 rounded bg-red-50" title="حذف"><i class="fa-solid fa-trash-can"></i></button>
                            <button onclick="selectStamp('${item.src}')" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold px-3 py-1.5 rounded-lg text-xs transition">
                                استخدام الختم
                            </button>
                        </div>
                    `;
                    container.appendChild(div);
                });
                return;
            }

            if (currentModalType === 'logo') {
                let list = localData.saved_logos || [];
                const filtered = list.filter(item => item.name.toLowerCase().includes(searchQuery));

                if (filtered.length === 0) {
                    container.innerHTML = '<p class="text-center text-slate-400 py-4">لا توجد شعارات محفوظة محلياً</p>';
                    return;
                }

                filtered.forEach((item, index) => {
                    const div = document.createElement('div');
                    div.className = "bg-slate-50 hover:bg-amber-50/50 p-2 rounded-xl border border-slate-200 flex justify-between items-center transition";
                    div.innerHTML = `
                        <div class="flex items-center gap-3 overflow-hidden">
                            <div class="w-12 h-12 bg-white border border-slate-200 rounded flex items-center justify-center shrink-0">
                                <img src="${item.src}" class="max-h-full max-w-full object-contain">
                            </div>
                            <p class="font-bold text-slate-900 truncate">${item.name}</p>
                        </div>
                        <div class="flex items-center gap-1 shrink-0">
                            <button onclick="deleteLogo(${index})" class="text-red-500 hover:text-red-700 p-1.5 rounded bg-red-50" title="حذف"><i class="fa-solid fa-trash-can"></i></button>
                            <button onclick="selectLogo('${item.src}')" class="bg-amber-500 hover:bg-amber-600 text-white font-bold px-3 py-1.5 rounded-lg text-xs transition">
                                اختيار
                            </button>
                        </div>
                    `;
                    container.appendChild(div);
                });
                return;
            }

            const key = currentModalType === 'seller' ? 'saved_sellers' : 'saved_buyers';
            let list = localData[key] || [];

            const filtered = list.filter(item => 
                item.name.toLowerCase().includes(searchQuery) || 
                item.vat.toLowerCase().includes(searchQuery)
            );

            if (filtered.length === 0) {
                container.innerHTML = '<p class="text-center text-slate-400 py-4">لا توجد بيانات محفوظة مطابقة للبحث</p>';
                return;
            }

            filtered.forEach((item, index) => {
                const div = document.createElement('div');
                div.className = "bg-slate-50 hover:bg-blue-50/50 p-3 rounded-xl border border-slate-200 flex justify-between items-center transition cursor-pointer";
                div.innerHTML = `
                    <div>
                        <p class="font-bold text-slate-900">${item.name}</p>
                        <p class="text-[11px] text-slate-500">الرقم الضريبي: <span class="font-mono">${item.vat}</span> | السجل: <span class="font-mono">${item.cr}</span></p>
                    </div>
                    <div class="flex items-center gap-2">
                        <button onclick="deleteParty('${currentModalType}', ${index})" class="text-rose-500 hover:text-rose-700 p-2 rounded-lg bg-rose-50 transition" title="حذف من الفهرس">
                            <i class="fa-solid fa-trash-can"></i>
                        </button>
                        <button onclick="selectParty('${currentModalType}', '${item.name}', '${item.vat}', '${item.cr}', '${item.address}')" class="bg-blue-600 hover:bg-blue-700 text-white font-bold px-3 py-1.5 rounded-lg text-xs transition">
                            اختيار
                        </button>
                    </div>
                `;
                container.appendChild(div);
            });
        }

        function selectParty(type, name, vat, cr, address) {
            document.getElementById(type + 'Name').innerText = name;
            document.getElementById(type + 'Vat').innerText = vat;
            document.getElementById(type + 'Cr').innerText = cr;
            document.getElementById(type + 'Address').innerText = address;
            
            if (type === 'seller') {
                renderTable();
            }
            syncInvoiceToReceipt();
            closeModal();
        }

        async function saveCurrentInvoice() {
            showLoading("جاري حفظ الفاتورة في الأرشيف المحلي...");
            
            const invNum = document.getElementById('stdInvNum').innerText.trim() || ('INV-' + Math.floor(Math.random()*1000));
            const invDate = document.getElementById('stdInvDate').innerText.trim();
            const buyerName = document.getElementById('buyerName').innerText.trim();
            const buyerVat = document.getElementById('buyerVat').innerText.trim();
            const buyerCr = document.getElementById('buyerCr').innerText.trim();
            const buyerAddress = document.getElementById('buyerAddress').innerText.trim();

            const sellerName = document.getElementById('sellerName').innerText.trim();
            const sellerVat = document.getElementById('sellerVat').innerText.trim();
            const sellerCr = document.getElementById('sellerCr').innerText.trim();
            const sellerAddress = document.getElementById('sellerAddress').innerText.trim();

            let logoSrc = document.getElementById('companyLogo').src;

            const payMethod = document.getElementById('paymentMethod').innerText.trim();
            const paidAmt = document.getElementById('paidDisplay').innerText.trim();
            const dueAmt = document.getElementById('dueDisplay').innerText.trim();
            const grandTotal = document.getElementById('grandTotalDisplay').innerText.trim();

            const uniqueId = 'INV_ID_' + Date.now();

            const invoiceData = {
                uniqueId: uniqueId,
                invNum: invNum,
                invDate: invDate,
                seller: { name: sellerName, vat: sellerVat, cr: sellerCr, address: sellerAddress },
                buyer: { name: buyerName, vat: buyerVat, cr: buyerCr, address: buyerAddress },
                items: JSON.parse(JSON.stringify(items)),
                logoSrc: logoSrc,
                payMethod: payMethod,
                paidAmt: paidAmt,
                dueAmt: dueAmt,
                grandTotal: grandTotal,
                template: currentTemplate,
                phase2Data: JSON.parse(JSON.stringify(phase2Data)),
                savedAt: new Date().toLocaleString('ar-SA')
            };

            if (!localData.saved_invoices) localData.saved_invoices = [];

            localData.saved_invoices.push(invoiceData);

            saveLocalStorageData();
            hideLoading();
            alert('تم حفظ الفاتورة رقم (' + invNum + ') في الأرشيف المحلي بنجاح!');
        }

        function openArchiveModal() {
            document.getElementById('archiveSearch').value = '';
            renderArchiveList('');
            document.getElementById('archiveModal').classList.remove('hidden');
        }

        function closeArchiveModal() {
            document.getElementById('archiveModal').classList.add('hidden');
        }

        function filterArchiveList() {
            const query = document.getElementById('archiveSearch').value.toLowerCase();
            renderArchiveList(query);
        }

        function renderArchiveList(query) {
            const container = document.getElementById('archiveListContainer');
            container.innerHTML = '';

            let archives = localData.saved_invoices || [];

            const filtered = archives.filter(inv => 
                (inv.buyer && inv.buyer.name && inv.buyer.name.toLowerCase().includes(query)) ||
                (inv.invNum && inv.invNum.toLowerCase().includes(query)) ||
                (inv.invDate && inv.invDate.toLowerCase().includes(query))
            );

            if (filtered.length === 0) {
                container.innerHTML = '<p class="text-center text-slate-400 py-6">لا توجد فواتير محفوظة مطابقة للبحث</p>';
                return;
            }

            filtered.forEach((inv, index) => {
                const div = document.createElement('div');
                div.className = "bg-slate-50 hover:bg-indigo-50/50 p-3 rounded-xl border border-slate-200 flex justify-between items-center transition mb-1";
                div.innerHTML = `
                    <div class="space-y-1">
                        <div class="flex items-center gap-2">
                            <span class="font-bold text-slate-900 text-sm">${inv.buyer ? inv.buyer.name : 'عميل بدون اسم'}</span>
                            <span class="bg-indigo-100 text-indigo-700 font-mono text-[11px] font-bold px-2 py-0.5 rounded-md">${inv.invNum}</span>
                        </div>
                        <p class="text-[11px] text-slate-500">التاريخ: <span class="font-mono">${inv.invDate}</span> | الإجمالي: <span class="font-bold font-mono text-emerald-600">${inv.grandTotal}</span></p>
                    </div>
                    <div class="flex items-center gap-2">
                        <button onclick="restoreInvoiceByIndex(${index})" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold px-3 py-1.5 rounded-lg text-xs transition flex items-center gap-1">
                            <i class="fa-solid fa-eye"></i> عرض / استرجاع
                        </button>
                        <button onclick="deleteInvoiceByIndex(${index})" class="bg-rose-100 hover:bg-rose-200 text-rose-600 font-bold p-1.5 rounded-lg text-xs transition" title="حذف">
                            <i class="fa-solid fa-trash-can"></i>
                        </button>
                    </div>
                `;
                container.appendChild(div);
            });
        }

        function restoreInvoiceByIndex(index) {
            let archives = localData.saved_invoices || [];
            const target = archives[index];

            if (!target) {
                alert('لم يتم العثور على الفاتورة!');
                return;
            }

            document.getElementById('stdInvNum').innerText = target.invNum || '';
            document.getElementById('tpl4InvNum').innerText = target.invNum || '';
            document.getElementById('stdInvDate').innerText = target.invDate || '';
            document.getElementById('tpl4InvDate').innerText = target.invDate || '';

            if (target.seller) {
                document.getElementById('sellerName').innerText = target.seller.name || '';
                document.getElementById('sellerVat').innerText = target.seller.vat || '';
                document.getElementById('sellerCr').innerText = target.seller.cr || '';
                document.getElementById('sellerAddress').innerText = target.seller.address || '';
            }

            if (target.buyer) {
                document.getElementById('buyerName').innerText = target.buyer.name || '';
                document.getElementById('buyerVat').innerText = target.buyer.vat || '';
                document.getElementById('buyerCr').innerText = target.buyer.cr || '';
                document.getElementById('buyerAddress').innerText = target.buyer.address || '';
            }

            if (target.logoSrc) document.getElementById('companyLogo').src = target.logoSrc;
            if (target.payMethod) document.getElementById('paymentMethod').innerText = target.payMethod;
            if (target.paidAmt) document.getElementById('paidDisplay').innerText = target.paidAmt;
            if (target.dueAmt) document.getElementById('dueDisplay').innerText = target.dueAmt;

            if (target.items && target.items.length > 0) {
                items = JSON.parse(JSON.stringify(target.items));
            }

            if (target.template) {
                switchTemplate(target.template);
            }

            if (target.phase2Data) {
                phase2Data = JSON.parse(JSON.stringify(target.phase2Data));
            }

            renderTable();
            syncInvoiceToReceipt();
            closeArchiveModal();
            alert('تم استرجاع الفاتورة (' + target.invNum + ') بنجاح!');
        }

        async function deleteInvoiceByIndex(index) {
            if (confirm('هل أنت متأكد من حذف هذه الفاتورة من الأرشيف المحلي؟')) {
                localData.saved_invoices.splice(index, 1);
                saveLocalStorageData();
                renderArchiveList(document.getElementById('archiveSearch').value.toLowerCase());
            }
        }

        function parseCurrency(str) {
            if (!str) return 0;
            const cleaned = str.replace(/[^\d.-]/g, '');
            return parseFloat(cleaned) || 0;
        }

        document.getElementById('paidDisplay').addEventListener('input', function() {
            const grandTotalStr = document.getElementById('grandTotalDisplay').innerText;
            const grandTotal = parseCurrency(grandTotalStr);
            const paidAmt = parseCurrency(this.innerText);
            
            const dueAmt = grandTotal - paidAmt;
            document.getElementById('dueDisplay').innerText = (dueAmt > 0 ? dueAmt : 0).toLocaleString('en-US', {minimumFractionDigits: 2}) + ' ر.س';
        });

        document.addEventListener('input', function(e) {
            if (e.target.classList.contains('editable-field')) {
                syncInvoiceToReceipt();
            }
        });

        window.onload = function() {
            loadLocalStorageData();
            checkUrlParamsAndApply();
            renderTable();
            switchTemplate(1);
        };
    </script>
</body>
</html>