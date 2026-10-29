# Ringkasan Hasil

- BM25 Dev nDCG@10 = 0.4486
- mBERT vs BM25: Δ nDCG@10 = +0.1462, 95% CI [+0.1245, +0.1678], Wilcoxon p(Holm) = 8.64e-36, lebih efektif = True
- mDeBERTa vs BM25: Δ nDCG@10 = +0.1550, 95% CI [+0.1340, +0.1759], Wilcoxon p(Holm) = 1.24e-41, lebih efektif = True
- mDeBERTa vs mBERT: Δ nDCG@10 = +0.0088, 95% CI [-0.0003, +0.0179], Wilcoxon p(Holm) = 1.83e-01, lebih efektif = False
- Latency k=100: mBERT 177.2 ms vs mDeBERTa 501.1 ms per kueri
- mBERT learning curve: 25% = 0.5492, 50% = 0.5725, 100% = 0.5947
- mDeBERTa learning curve: 25% = 0.5715, 50% = 0.5891, 100% = 0.6035