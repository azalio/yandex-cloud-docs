In [asynchronous recognition](../../speechkit/stt/transcribation.md), a language model is available that can work in two modes:
1. In standard mode, recognition is processed in a queue with a standard priority. The mode works when the `general` model is selected.
1. In deferred mode, the audio file for recognition gets into the queue with a low priority and is processed at the least busy time. Deferred recognition is subject to [special rates](../../speechkit/pricing.md#prices-stt). The processing time of an audio file in deferred mode is no more than 24 hours. Recognition in deferred mode is available when the `deferred-general` tag is specified.