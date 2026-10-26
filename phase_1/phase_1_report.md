Your notebook is a solid baseline: it splits whole tracks before making windows, keeps exact file copies in the same split, and scores predictions at the track level. The main opportunities are **better evaluation, transfer learning, and more varied training examples**—in that order.

### What I’d change first

1. **Make the score more reliable.** The notebook tests on one roughly 10% split, so 67% may reflect the particular tracks in that split. If it is about 100 tracks, that is roughly 67 correct; a rough 95% interval is about 57–75%, before accounting for split variation. Repeat evaluation across several group-aware folds and report the mean and spread. Also, the current training metric is *window* accuracy while validation is *track* accuracy, so they can’t be compared directly to diagnose overfitting.

2. **Split by artist or recording, not only exact file hash.** Your hash grouping prevents exact copies crossing splits, but it does not catch different tracks by the same artist, alternate versions, or other near-duplicates. GTZAN’s documented issues include repeated recordings and artists, along with mislabeled examples. The dataset audit and its associated repository provide an artist/title index and fault-filtered partitions you can use to make evaluation more defensible. [Sturm’s GTZAN analysis](https://projet.liris.cnrs.fr/imagine/pub/proceedings/ACM-MULTIMEDIA-2012/mirum/p7.pdf), [GTZAN partitions and index](https://github.com/boblsturm/GTZAN)

3. **Try a pretrained audio model before making the CNN deeper.** With only about 900 training tracks, learning all audio features from scratch is a major constraint. PANNs’ GTZAN experiments reported 75.8% for training CNN14 from scratch, 85.8% using a frozen feature model with a deeper head, and 91.5% after fine-tuning, using 10-fold evaluation. Those figures are **not directly comparable** to your single split, but they make fine-tuning a strong experiment. Start with a frozen PANNs encoder and a small classifier; then try fine-tuning with a lower learning rate for the encoder. Follow the pretrained model’s required sample rate and spectrogram preprocessing rather than feeding it your current cached mel features unchanged. [PANNs paper](https://arxiv.org/abs/1912.10211)

   A recent study also reports 81.5% using BYOL-A embeddings and a trained classifier, versus 77% with PANNs embeddings in that study. Treat those as leads to test under your own split protocol, not guaranteed targets. [BYOL-A/PANNs/VGGish comparison](https://arxiv.org/abs/2603.13871)

4. **Add variety to the training crops.** Right now each track produces the same six non-overlapping five-second windows every time. Randomize crop positions during training, and try mild frequency/time masking (SpecAugment) and mixup. Keep augmentation confined to training. At inference, compare your current six-window average with overlapping crops across the track.

5. **Only then tune the small CNN.** Your model globally averages the final feature maps down to one value per channel, which can discard local time/frequency patterns. Try a residual CNN or preserve a small spatial grid before the classifier. Tune dropout, weight decay, and learning rate using validation folds. I would not expect simply adding layers to reliably help on this dataset.

### A practical experiment order

Keep the current model as a baseline. First measure it over repeated artist-aware folds; then compare a frozen pretrained encoder, fine-tuning, and finally augmentation or architecture changes. Record track-level accuracy and macro-F1 for every fold. This will show whether a change improves the model consistently rather than just one 100-track split.

The larger reported GTZAN scores in the literature often use different partitions and evaluation setups, so compare methods only when the data split and scoring protocol match.
