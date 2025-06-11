<script>
    import { onMount } from "svelte";
    import { browser } from "$app/environment";

    let html2canvasPro;
    let jsPDF;
    let companyName = "",
        bizNumber = "",
        address = "",
        ceo = "",
        contactPerson = "",
        email = "",
        selectedPackage = "";

    // 공급하는 자 정보 (주최자 정보)
    let supplierInfo = {
        companyName: import.meta.env.VITE_COMPANY_NAME,
        bizNumber: import.meta.env.VITE_BIZ_NUMBER,
        address: import.meta.env.VITE_ADDRESS,
        ceo: import.meta.env.VITE_CEO,
        contactPerson: "",
        phone: "",
        email: "",
    };

    let packages = [
        {
            id: "bronze",
            name: "브론즈",
            amount: 800000,
            benefits: [
                "로고 노출 (기본)",
                "입장권 지원 (2장 무료)",
                "마케팅 (기본)",
                "마케팅 (이메일 - 이름, 링크 및 로고 삽입, 상호작용)",
                "후원사의 증정품 지급 가능",
            ],
        },
        {
            id: "silver",
            name: "실버",
            amount: 1500000,
            benefits: [
                "로고 노출 (브론즈보다 크게)",
                "입장권 4장 무료",
                "기본 마케팅",
                "이메일 마케팅",
                "SNS 마케팅",
                "증정품 지급 가능",
            ],
        },
        {
            id: "gold",
            name: "골드",
            amount: 3000000,
            benefits: [
                "메인 로고 노출",
                "입장권 8장 무료",
                "프리미엄 마케팅",
                "이메일 마케팅",
                "SNS 마케팅",
                "현장 부스 운영",
                "증정품 지급 가능",
            ],
        },
    ];

    let isGenerating = false;

    // 클라이언트 사이드에서만 라이브러리 로드
    onMount(async () => {
        if (browser) {
            try {
                // html2canvas-pro와 jsPDF 동적 로드
                const [html2canvasModule, jsPDFModule] = await Promise.all([
                    import("html2canvas-pro"),
                    import("jspdf"),
                ]);

                html2canvasPro = html2canvasModule.default;
                jsPDF = jsPDFModule.jsPDF;

                console.log("라이브러리 로드 완료");
            } catch (error) {
                console.error("라이브러리 로드 실패:", error);
            }
        }
    });

    async function handleDownloadPDF() {
        if (!browser || !html2canvasPro || !jsPDF) {
            alert("PDF 생성 라이브러리가 아직 로드되지 않았습니다.");
            return;
        }

        if (!companyName || !selectedPackage) {
            alert("회사명과 후원 패키지를 선택해주세요.");
            return;
        }

        const element = document.getElementById("pdf-content");
        if (!element) {
            console.error("PDF 컨텐츠 요소를 찾을 수 없습니다.");
            return;
        }

        isGenerating = true;

        try {
            // html2canvas-pro 옵션 설정
            const canvas = await html2canvasPro(element, {
                scale: 2, // 고화질
                useCORS: true,
                allowTaint: true,
                backgroundColor: "#ffffff",
                logging: false,
                letterRendering: true,
                width: 794, // A4 width in pixels at 96 DPI (210mm)
                height: 1123, // A4 height in pixels at 96 DPI (297mm)
                windowWidth: 794,
                windowHeight: 1123,
                imageTimeout: 0,
                removeContainer: true,
            });

            // jsPDF로 PDF 생성
            const imgData = canvas.toDataURL("image/jpeg", 1.0);
            const pdf = new jsPDF({
                orientation: "portrait",
                unit: "mm",
                format: "a4",
            });

            // A4 크기로 정확히 맞춤
            const pdfWidth = pdf.internal.pageSize.getWidth();
            const pdfHeight = pdf.internal.pageSize.getHeight();

            // PDF에 이미지를 A4 크기에 정확히 맞춰 추가
            pdf.addImage(imgData, "JPEG", 0, 0, pdfWidth, pdfHeight);

            // PDF 저장
            pdf.save(`${companyName}_견적서.pdf`);
        } catch (error) {
            console.error("PDF 생성 중 오류 발생:", error);
            alert("PDF 생성 중 오류가 발생했습니다. 다시 시도해주세요.");
        } finally {
            isGenerating = false;
        }
    }

    // 선택된 패키지 정보 가져오기
    $: selectedPackageInfo = packages.find((p) => p.id === selectedPackage);
    $: isReady = browser && html2canvasPro && jsPDF;
    $: totalAmount = selectedPackageInfo ? selectedPackageInfo.amount : 0;
    $: vatAmount = Math.floor(totalAmount * 0.1);
    $: totalWithVat = totalAmount + vatAmount;
</script>

<div class="min-h-screen bg-gray-50 p-6 overflow-x-auto">
    <div class="max-w-7xl mx-auto">
        <h1 class="text-3xl font-bold text-center mb-8 text-gray-800">
            후원 견적서 생성기
        </h1>

        <div class="grid grid-cols-1 xl:grid-cols-5 gap-5">
            <!-- 왼쪽: 입력 폼 -->
            <div class="xl:col-span-2 bg-white rounded-lg shadow-lg p-6">
                <h2 class="text-xl font-semibold mb-6 text-gray-700">
                    담당하는 자 정보 입력
                </h2>

                <form class="space-y-4 pb-8">
                    <div>
                        <label
                            for="supplierInfoContactPerson"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            담당자
                        </label>
                        <input
                            id="supplierInfoContactPerson"
                            bind:value={supplierInfo.contactPerson}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="담당자명을 입력하세요"
                        />
                    </div>

                    <div>
                        <label
                            for="supplierInfoPhone"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            담당자 전화번호
                        </label>
                        <input
                            id="supplierInfoPhone"
                            bind:value={supplierInfo.phone}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="010-0000-0000"
                        />
                    </div>

                    <div>
                        <label
                            for="supplierInfoEmail"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            담당자 전화번호
                        </label>
                        <input
                            id="supplierInfoEmail"
                            bind:value={supplierInfo.email}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="contact@company.com"
                        />
                    </div>
                </form>
                <h2 class="text-xl font-semibold mb-6 text-gray-700">
                    공급받는 자 정보 입력
                </h2>

                <form class="space-y-4">
                    <div>
                        <label
                            for="companyName"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            회사명 *
                        </label>
                        <input
                            id="companyName"
                            bind:value={companyName}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="회사명을 입력하세요"
                        />
                    </div>

                    <div>
                        <label
                            for="bizNumber"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            사업자등록번호
                        </label>
                        <input
                            id="bizNumber"
                            bind:value={bizNumber}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="000-00-00000"
                        />
                    </div>

                    <div>
                        <label
                            for="address"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            주소
                        </label>
                        <input
                            id="address"
                            bind:value={address}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="회사 주소를 입력하세요"
                        />
                    </div>

                    <div>
                        <label
                            for="ceo"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            대표자명
                        </label>
                        <input
                            id="ceo"
                            bind:value={ceo}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="대표자 이름을 입력하세요"
                        />
                    </div>

                    <div>
                        <label
                            for="contactPerson"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            담당자명
                        </label>
                        <input
                            id="contactPerson"
                            bind:value={contactPerson}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="담당자 이름을 입력하세요"
                        />
                    </div>

                    <div>
                        <label
                            for="email"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            이메일
                        </label>
                        <input
                            id="email"
                            type="email"
                            bind:value={email}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                            placeholder="contact@company.com"
                        />
                    </div>

                    <div>
                        <label
                            for="package"
                            class="block text-sm font-medium text-gray-700 mb-1"
                        >
                            후원 패키지 *
                        </label>
                        <select
                            id="package"
                            bind:value={selectedPackage}
                            class="w-full border border-gray-300 rounded-md shadow-sm px-3 py-2 bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                        >
                            <option value="">-- 패키지를 선택하세요 --</option>
                            {#each packages as p}
                                <option value={p.id}>
                                    {p.name} - ₩{p.amount.toLocaleString()}
                                </option>
                            {/each}
                        </select>
                    </div>

                    <button
                        type="button"
                        on:click={handleDownloadPDF}
                        class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-gray-400 text-white font-semibold py-3 px-4 rounded-md shadow transition-colors flex items-center justify-center gap-2"
                        disabled={!isReady ||
                            !companyName ||
                            !selectedPackage ||
                            isGenerating}
                    >
                        {#if isGenerating}
                            <svg
                                class="animate-spin h-5 w-5"
                                xmlns="http://www.w3.org/2000/svg"
                                fill="none"
                                viewBox="0 0 24 24"
                            >
                                <circle
                                    class="opacity-25"
                                    cx="12"
                                    cy="12"
                                    r="10"
                                    stroke="currentColor"
                                    stroke-width="4"
                                ></circle>
                                <path
                                    class="opacity-75"
                                    fill="currentColor"
                                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                                ></path>
                            </svg>
                            PDF 생성 중...
                        {:else if !isReady}
                            라이브러리 로딩 중...
                        {:else}
                            📄 PDF 다운로드
                        {/if}
                    </button>

                    {#if !isReady}
                        <p class="text-sm text-gray-500 text-center">
                            PDF 생성 라이브러리를 로드하고 있습니다...
                        </p>
                    {/if}
                </form>
            </div>

            <!-- 오른쪽: 견적서 미리보기 -->
            <div class="xl:col-span-3 bg-white rounded-lg shadow-lg">
                <div
                    id="pdf-content"
                    class="p-8 bg-white min-h-[297mm]"
                    style="font-family: 'Malgun Gothic', sans-serif; width: 210mm; max-width: none;"
                >
                    <div class="border-b-2 border-blue-600 pb-4 mb-6">
                        <h1
                            class="text-3xl font-bold text-center text-gray-800"
                        >
                            견적서
                        </h1>
                        <p class="text-center text-gray-600 mt-2">Quotation</p>
                    </div>

                    <!-- 공급자 & 공급받는자 정보 -->
                    <div class="grid grid-cols-2 gap-6 mb-8">
                        <!-- 공급자 정보 -->
                        <div class="border rounded-lg p-4">
                            <h2
                                class="text-lg font-semibold mb-4 text-gray-700 bg-blue-50 px-3 py-2 rounded"
                            >
                                공급자
                            </h2>
                            <div class="space-y-2 text-sm">
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >사업자등록번호:</span
                                    >
                                    <span class="text-gray-800"
                                        >{supplierInfo.bizNumber}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >회사명:</span
                                    >
                                    <span class="text-gray-800 font-semibold"
                                        >{supplierInfo.companyName}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >주소:</span
                                    >
                                    <span class="text-gray-800"
                                        >{supplierInfo.address}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >대표:</span
                                    >
                                    <span class="text-gray-800"
                                        >{supplierInfo.ceo}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >담당자:</span
                                    >
                                    <span class="text-gray-800"
                                        >{supplierInfo.contactPerson}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >전화:</span
                                    >
                                    <span class="text-gray-800"
                                        >{supplierInfo.phone}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >이메일:</span
                                    >
                                    <span class="text-gray-800"
                                        >{supplierInfo.email}</span
                                    >
                                </div>
                            </div>
                        </div>

                        <!-- 공급받는자 정보 -->
                        <div class="border rounded-lg p-4">
                            <h2
                                class="text-lg font-semibold mb-4 text-gray-700 bg-green-50 px-3 py-2 rounded"
                            >
                                공급받는자
                            </h2>
                            <div class="space-y-2 text-sm">
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >사업자등록번호:</span
                                    >
                                    <span class="text-gray-800"
                                        >{bizNumber || "미입력"}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >회사명:</span
                                    >
                                    <span class="text-gray-800 font-semibold"
                                        >{companyName || "미입력"}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >주소:</span
                                    >
                                    <span class="text-gray-800"
                                        >{address || "미입력"}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >대표:</span
                                    >
                                    <span class="text-gray-800"
                                        >{ceo || "미입력"}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >담당자:</span
                                    >
                                    <span class="text-gray-800"
                                        >{contactPerson || "미입력"}</span
                                    >
                                </div>
                                <div class="flex flex-col">
                                    <span class="font-medium text-gray-600"
                                        >이메일:</span
                                    >
                                    <span class="text-gray-800"
                                        >{email || "미입력"}</span
                                    >
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- 견적 상세 내역 -->
                    {#if selectedPackageInfo}
                        <div class="mb-8">
                            <h2
                                class="text-lg font-semibold mb-4 text-gray-700 border-l-4 border-blue-500 pl-3"
                            >
                                상세 내역
                            </h2>
                            <div class="border rounded-lg overflow-hidden">
                                <table class="w-full">
                                    <thead class="bg-gray-50">
                                        <tr>
                                            <th
                                                class="px-4 py-3 text-left text-sm font-medium text-gray-700"
                                                >품목</th
                                            >
                                            <th
                                                class="px-4 py-3 text-center text-sm font-medium text-gray-700"
                                                >단가</th
                                            >
                                            <th
                                                class="px-4 py-3 text-center text-sm font-medium text-gray-700"
                                                >수량</th
                                            >
                                            <th
                                                class="px-4 py-3 text-center text-sm font-medium text-gray-700"
                                                >공급가액</th
                                            >
                                            <th
                                                class="px-4 py-3 text-center text-sm font-medium text-gray-700"
                                                >세액</th
                                            >
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr class="border-t">
                                            <td class="px-4 py-4 text-sm">
                                                <div
                                                    class="font-medium text-gray-800"
                                                >
                                                    UbuCon Korea 2025 {selectedPackageInfo.name}
                                                    후원사 패키지
                                                </div>
                                                <div
                                                    class="mt-2 text-xs text-gray-600"
                                                >
                                                    {#each selectedPackageInfo.benefits as benefit}
                                                        <div>- {benefit}</div>
                                                    {/each}
                                                </div>
                                            </td>
                                            <td
                                                class="px-4 py-4 text-sm text-center"
                                                >₩{selectedPackageInfo.amount.toLocaleString()}</td
                                            >
                                            <td
                                                class="px-4 py-4 text-sm text-center"
                                                >1 EA</td
                                            >
                                            <td
                                                class="px-4 py-4 text-sm text-center"
                                                >₩{selectedPackageInfo.amount.toLocaleString()}</td
                                            >
                                            <td
                                                class="px-4 py-4 text-sm text-center"
                                                >₩{vatAmount.toLocaleString()}</td
                                            >
                                        </tr>
                                    </tbody>
                                    <tfoot class="bg-gray-50 font-medium">
                                        <tr>
                                            <td
                                                colspan="3"
                                                class="px-4 py-3 text-sm text-right"
                                                >합계</td
                                            >
                                            <td
                                                class="px-4 py-3 text-sm text-center"
                                                >₩{selectedPackageInfo.amount.toLocaleString()}</td
                                            >
                                            <td
                                                class="px-4 py-3 text-sm text-center"
                                                >₩{vatAmount.toLocaleString()}</td
                                            >
                                        </tr>
                                        <tr>
                                            <td
                                                colspan="4"
                                                class="px-4 py-3 text-sm text-right font-bold"
                                                >합계 (부가세 포함)</td
                                            >
                                            <td
                                                class="px-4 py-3 text-sm text-center font-bold text-blue-600"
                                                >₩{totalWithVat.toLocaleString()}</td
                                            >
                                        </tr>
                                    </tfoot>
                                </table>
                            </div>
                        </div>
                    {:else}
                        <div class="mb-8">
                            <h2
                                class="text-lg font-semibold mb-4 text-gray-700 border-l-4 border-gray-300 pl-3"
                            >
                                상세 내역
                            </h2>
                            <div
                                class="bg-gray-50 rounded-lg p-4 text-center text-gray-500"
                            >
                                패키지를 선택하면 여기에 상세 내역이 표시됩니다
                            </div>
                        </div>
                    {/if}

                    <div class="border-t pt-4 mt-8">
                        <div
                            class="grid grid-cols-2 gap-4 text-sm text-gray-600"
                        >
                            <div>
                                <span class="font-medium">견적일자:</span>
                                {new Date().toLocaleDateString("ko-KR")}
                            </div>
                            <div>
                                <span class="font-medium">견적 유효기간:</span> 견적서
                                발행일로부터 2주
                            </div>
                        </div>
                        <div class="mt-4 text-center">
                            <p class="text-sm text-gray-600">
                                위와 같이 견적합니다.
                            </p>
                            <div class="mt-2 text-right">
                                <span class="text-sm text-gray-600"
                                    >(대표자 서명 또는 인)</span
                                >
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
