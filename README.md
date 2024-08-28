- 👋 Hi, I’m @Kevin97B
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Kevin97B/Kevin97B is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Für Lizenz- und Copyrightinformationen folgen Sie bitte diesem Link:
https://github.com/telegramdesktop/tdesktop/blob/master/LEGAL
*/
# include  " history/view/media/history_view_web_page.h "
# include  " core/application.h "
# include  " Länder/Länder_Instanz.h "
# include  " base/qt/qt_key_modifiers.h "
# include  " window/window_session_controller.h "
# include  " iv/iv_instance.h "
# include  " core/click_handler_types.h "
# include  " core/ui_integration.h "
# include  " data/components/sponsored_messages.h "
# include  " data/stickers/data_custom_emoji.h "
# include  " data/data_file_click_handler.h "
# include  " data/data_photo_media.h "
# include  " data/data_session.h "
# include  " data/data_web_page.h "
# include  " history/view/media/history_view_media_common.h "
# include  " history/view/media/history_view_sticker.h "
# include  " history/view/history_view_cursor_state.h "
# include  " history/view/history_view_message.h "
# include  " history/view/history_view_reply.h "
# include  " history/view/history_view_sponsored_click_handler.h "
# include  " history/history.h "
# include  " history/history_item_components.h "
# include  " lang/lang_keys.h "
# include  " main/main_session.h "
# include  " menu/menu_sponsored.h "
# include  " ui/chat/chat_style.h "
# include  " ui/painter.h "
# include  " ui/rect.h "
# include  " ui/power_saving.h "
# include  " ui/text/format_values.h "
# include  " ui/text/text_options.h "
# include  " ui/text/text_utilities.h "
# include  " ui/toast/toast.h "
# include  " styles/style_chat.h "
Namespace  Verlaufsansicht {
Namensraum {
constexpr  auto  kMaxOriginalEntryLines = 8192 ;
constexpr  auto  kFactcheckCollapsedLines = 3 ;
constexpr  auto  kStickerSetLines = 3 ;
constexpr  auto  kFactcheckAboutDuration = 5 * crl::time ( 1000 );
[[nodiscard]] int  ArticleThumbWidth (not_null<PhotoData*> Daumen, int Höhe) {
	const  auto size = Daumen-> Standort (Daten::Fotogröße::Thumbnail);
	gibt Größe zurück . Höhe ()
		? std::max ( std::min ( Höhe * Größe.Breite () / Größe.Höhe ( ), Höhe), 1 )
		: 1 ;
}
[[nodiscard]] int  ArticleThumbHeight (
		not_null<Data::PhotoMedia*> Daumen,
		int Breite) {
	const  auto size = Daumen-> Größe (Daten::Fotogröße::Thumbnail);
	gibt Größe zurück . Breite ()
		? std::max (Größe.Höhe ( ) * Breite / Größe.Breite (), 1 )
		: 1 ;
}
[[nodiscard]] std::vector<std::unique_ptr<Data::Media>> PrepareCollageMedia (
		not_null<HistoryItem*> übergeordnetes Element,
		const WebPageCollage &data) {
	automatisches Ergebnis = std::vector<std::unique_ptr<Data::Media>>();
	Ergebnis. Reserve (Daten. Elemente . Größe ());
	const  auto spoiler = false ;
	für ( const  auto &item: data.items ) {
		wenn ( const  auto document = std::get_if<DocumentData*>(&item)) {
			const  auto skipPremiumEffect = false ;
			Ergebnis. Push_back (std::make_unique<Data::MediaFile>(
				Elternteil,
				*dokumentieren,
				überspringenPremiumEffect,
				Spoiler,
				/* ttlSekunden = */ 0 ));
		} sonst  wenn ( const  auto photo = std::get_if<PhotoData*>(&item)) {
			Ergebnis. Push_back (std::make_unique<Data::MediaPhoto>(
				Elternteil,
				*Foto,
				Spoiler));
		} anders {
			zurückkehren {};
		}
		wenn (!Ergebnis.zurück ()-> kannGruppiert Werden ( )) {
			zurückkehren {};
		}
	}
	Ergebnis zurückgeben ;
}
[[nodiscard]] QString ExtractHash (
		not_null<WebPageData*> Webseite,
		const TextWithEntities &text) {
	const  auto vereinfachen = []( const QString &url) {
		automatisches Ergebnis = URL.Split ( ' # ' )[ 0 ] .toLower ( );
		wenn (Ergebnis. endet mit ( ' / ' )) {
			Ergebnis. hacken ( 1 );
		}
		const  auto Präfixe = { u" http:// " _q, u" https:// " _q };
		für ( const  auto &prefix : Präfixe) {
			if (Ergebnis.startetMit ( Präfix )) {
				Ergebnis = Ergebnis.mitte (Präfix.größe ( ) );
				brechen ;
			}
		}
		Ergebnis zurückgeben ;
	};
	const  auto vereinfacht = vereinfachen (Webseite -> URL );
	für ( const  auto &entity: text.entities ) {
		const  auto  link = (entity.typ ( ) == EntityType::Url)
			? Text.Text.Mitte ( Entität.Offset ( ) , Entität.Länge ( ) )
			: (Entität.Typ ( ) == EntityType::CustomUrl)
			? Entität.Daten ( )
			: QString ();
		wenn ( vereinfachen ( Link ) == vereinfacht) {
			const  auto i = link.indexOf ( ' # ' ) ;
			Rückgabe (i > 0 )? Link . Mitte (i + 1 ): QString ();
		}
	}
	 gibt QString () zurück ;
}
[[nodiscard]] ClickHandlerPtr IvClickHandler (
		not_null<WebPageData*> Webseite,
		const TextWithEntities &text) {
	return std::make_shared<LambdaClickHandler>([=](ClickContext-Kontext) {
		const  auto my = Kontext. Anderes . Wert <ClickHandlerContext>();
		wenn ( const  auto controller = mein.sessionWindow.get ( ) ) {
			wenn ( const  auto iv = Webseite-> iv . get ()) {
				const  auto hash = ExtractHash (Webseite, Text);
				Core::App ().iv ( ) .zeige (Controller, iv, Hash);
				zurückkehren ;
			} anders {
				HiddenUrlClickHandler::Open (Webseite -> URL , Kontext. Sonstiges );
			}
		}
	});
}
[[nodiscard]] ClickHandlerPtr ÜberSponsoredClickHandler () {
	return std::make_shared<LambdaClickHandler>([=](ClickContext-Kontext) {
		const  auto my = Kontext. Anderes . Wert <ClickHandlerContext>();
		wenn ( const  auto controller = mein.sessionWindow.get ( ) ) {
			Menü::ShowSponsoredAbout (Controller-> uiShow ());
		}
	});
}
[[nodiscard]] QString LookupFactcheckCountryIso2 (
		not_null<HistoryItem*> Element) {
	const  auto info = item-> Get <HistoryMessageFactcheck>();
	Informationen zurückgeben ? Info-> Daten . Land : QString ();
}
[[nodiscard]] QString LookupFactcheckCountryName ( const QString &iso2) {
	const  auto name = Länder::Instanz ().countryNameByISO2 ( iso2);
	gibt den Namen zurück . isEmpty () ? iso2 : Name;
}
[[nodiscard]] ClickHandlerPtr AboutFactcheckClickHandler (QString iso2) {
	return std::make_shared<LambdaClickHandler>([=](ClickContext-Kontext) {
		const  auto my = Kontext. Anderes . Wert <ClickHandlerContext>();
		const  auto controller = mein.sessionWindow.get ( ) ;
		const  auto show = meine.show
			? meine. Show
			: Controller
			? Controller-> uiShow ()
			: nullptr ;
		wenn (anzeigen) {
			const  auto country = LookupFactcheckCountryName (iso2);
			anzeigen-> showToast ({
				. text = {
					tr::lng_factcheck_about (tr::jetzt, lt_land, Land)
				},
				. Dauer = kFactcheckAboutDuration ,
			});
		}
	});
}
[[nodiscard]] ClickHandlerPtr ToggleFactcheckClickHandler (
		not_null<Element*> Ansicht) {
	const  auto weak = base::make_weak (Ansicht);
	return std::make_shared<LambdaClickHandler>([=](ClickContext-Kontext) {
		wenn ( const  auto stark = schwach.get ( )) {
			wenn ( const  auto factcheck = strong-> Holen Sie sich <Factcheck>()) {
				factcheck-> erweitert = factcheck-> erweitert ? 0 : 1 ;
				strong-> Verlauf ()-> Eigentümer (). requestViewResize (strong);
			}
		}
	});
}
[[nodiscard]] TextWithEntities PageToPhrase (not_null<WebPageData*> Seite) {
	const  auto Typ = Seite-> Typ ;
	const  auto text = Ui::Text::Upper (Seite-> iv
		? tr::lng_view_button_iv (tr::jetzt)
		: (Typ == Webseitentyp::Design)
		? tr::lng_view_button_theme (tr::jetzt)
		: (Typ == Webseitentyp::Story)
		? tr::lng_view_button_story (tr::jetzt)
		: (Typ == Webseitentyp::Nachricht)
		? tr::lng_view_button_message (tr::jetzt)
		: (Typ == Webseitentyp::Gruppe)
		? tr::lng_view_button_group (tr::jetzt)
		: (Typ == Webseitentyp::Hintergrundbild)
		? tr::lng_view_button_background (tr::jetzt)
		: (Typ == Webseitentyp::Kanal)
		? tr::lng_view_button_channel (tr::jetzt)
		: (Typ == Webseitentyp::GruppeMitAnforderung
			|| Typ == Webseitentyp::KanalMitAnforderung)
		? tr::lng_view_button_request_join (tr::jetzt)
		: (Typ == Webseitentyp::GroupBoost
			|| Typ == WebPageType::ChannelBoost)
		? tr::lng_view_button_boost (tr::jetzt)
		: (Typ == Webseitentyp::Geschenkcode)
		? tr::lng_view_button_giftcode (tr::jetzt)
		: (Typ == Webseitentyp::VoiceChat)
		? tr::lng_view_button_voice_chat (tr::jetzt)
		: (Typ == Webseitentyp::Livestream)
		? tr::lng_view_button_voice_chat_channel (tr::jetzt)
		: (Typ == Webseitentyp::Bot)
		? tr::lng_view_button_bot (tr::jetzt)
		: (Typ == Webseitentyp::Benutzer)
		? tr::lng_view_button_user (tr::jetzt)
		: (Typ == Webseitentyp::BotApp)
		? tr::lng_view_button_bot_app (tr::jetzt)
		: (Seite-> StickerSet && Seite-> StickerSet -> istEmoji )
		? tr::lng_view_button_emojipack (tr::jetzt)
		: (Typ == Webseitentyp::StickerSet)
		? tr::lng_view_button_stickerset (tr::jetzt)
		: QString ());
	if (seite-> iv ) {
		const  auto manager = &page-> Besitzer () .customEmojiManager ();
		const  auto &icon = st::historyIvIcon;
		const  auto padding = st::historyIvIconPadding;
		zurückgeben  Ui::Text::SingleCustomEmoji (
			Manager-> registerInternalEmoji (Symbol, Polsterung)
		). anhängen (Text);
	}
	Rückgabewert { Text };
}
[[nodiscard]] bool  HasButton (not_null<WebPageData*> Webseite) {
	const  auto Typ = Webseite-> Typ ;
	Webseite zurückgeben -> iv
		|| (Typ == Webseitentyp::Nachricht)
		|| (Typ == Webseitentyp::Gruppe)
		|| (Typ == Webseitentyp::GruppeMitAnforderung)
		|| (Typ == Webseitentyp::GroupBoost)
		|| (Typ == Webseitentyp::Kanal)
		|| (Typ == Webseitentyp::ChannelBoost)
		|| (Typ == Webseitentyp::KanalMitAnforderung)
		|| (Typ == Webseitentyp::Geschenkcode)
		// || (Typ == Webseitentyp::Bot)
		|| (Typ == Webseitentyp::Benutzer)
		|| (Typ == Webseitentyp::VoiceChat)
		|| (Typ == Webseitentyp::Livestream)
		|| (Typ == Webseitentyp::BotApp)
		|| ((Typ == Webseitentyp::Design)
			&& Webseite-> Dokument
			&& Webseite -> Dokument -> isTheme ())
		|| ((Typ == Webseitentyp::Story)
			&& (Webseite-> Foto || Webseite-> Dokument ))
		|| ((Typ == Webseitentyp::WallPaper)
			&& Webseite-> Dokument
			&& Webseite-> Dokument -> isWallPaper ())
		|| (Typ == Webseitentyp::StickerSet);
}
} // Namensraum
Webseite::Webseite (
	not_null<Element*> übergeordnetes Element,
	not_null<WebPageData*> Daten,
	MediaWebPageFlags-Flags)
: Medien (übergeordnet)
, _st(Daten->Typ == Webseitentyp::Factcheck
	? st::factcheckPage
	: st::historyPagePreview)
, _data(Daten)
, _flags(Flags)
, _siteName(st::msgMinWidth - _st.padding.left() - _st.padding.right())
, _title(st::msgMinWidth - _st.padding.left() - _st.padding.right())
, _Beschreibung(st::msgMinWidth - _st.padding.left() - _st.padding.right()) {
	Verlauf () -> Eigentümer ().registerWebPageView ( _data, _parent);
}
void  Webseite::setupAdditionalData () {
	if (_flags & MediaWebPageFlag::Sponsored) {
		_additionalData = std::make_unique<AdditionalData>( SponsoredData ());
		const  auto raw = gesponserteDaten ();
		const  auto &session = _parent-> Daten ()-> Verlauf ()-> Sitzung ();
		const  auto details = Sitzung.gesponserteNachrichten ( ) .lookupDetails (
			_parent-> Daten ()-> fullId ());
		raw-> buttonText = Details. buttonText ;
		raw-> isLinkInternal = Details. isLinkInternal ? 1 : 0 ;
		raw-> backgroundEmojiId = Details. backgroundEmojiId ;
		raw-> Farbindex = Details. Farbindex ;
		raw-> canReport = Details. canReport ? 1 : 0 ;
	} sonst  wenn (_data-> stickerSet ) {
		_additionalData = std::make_unique<AdditionalData>( StickerSetData ());
		const  auto raw = stickerSetData ();
		für ( const  auto &sticker: _data-> stickerSet -> items ) {
			wenn (!aufkleber-> aufkleber ()) {
				weitermachen ;
			}
			raw-> Ansichten . push_back (
				std::make_unique<Aufkleber>(_parent, Aufkleber, true ));
		}
		const  auto side = std::ceil ( std::sqrt (raw-> Ansichten . Größe ()));
		const  auto box = UnitedLineHeight () * kStickerSetLines ;
		const  auto single = Box / Seite;
		für ( const  auto &view: raw-> Ansichten ) {
			Ansicht -> setzeWebseitenteil ();
			Ansicht -> InitSize (einzeln);
		}
	} sonst  wenn (_data-> Typ == WebPageType::Factcheck) {
		_additionalData = std::make_unique<AdditionalData>( FactcheckData ());
	}
}
QSize Webseite::countOptimalSize () {
	wenn (_data-> pendingTill || _data-> fehlgeschlagen ) {
		Rückgabewert { 0 , 0 };
	}
	setupAdditionalData ();
	const  auto gesponsert = gesponserteDaten ();
	const  auto factcheck = factcheckData ();
	// Erkennen Sie _openButtonWidth, bevor Sie die Polsterungen zählen.
	_openButton = Ui::Text::String ();
	wenn ( HasButton (_data)) {
		const  auto context = Core::MarkierteTextContext{
			. Sitzung = &_Daten-> Sitzung (),
			.customEmojiRepaint = [] {},
			.customEmojiLoopLimit = 1 ,​
		};
		_openButton.setMarkedText (​
			st::halbfetterTextstil,
			PageToPhrase (_Daten),
			kMarkupTextOptions ,
			Kontext);
	} sonst  wenn (gesponsert und !sponsored-> buttonText . isEmpty ()) {
		_openButton.setText (​
			st::halbfetterTextstil,
			Ui::Text::Upper (gesponsert-> buttonText ));
	}
	const  auto padding = inBubblePadding () + innerMargin ();
	const  auto versionChanged = (_dataVersion != _data-> version );
	if (versionGeändert) {
		_dataVersion = _data-> Version ;
		_openl = nullptr ;
		_attach = nullptr ;
		const  auto item = _parent-> data ();
		_collage = PrepareCollageMedia (Element, _Daten-> Collage );
		const  auto min = st::msgMinWidth - rect::m::sum::h (_st. padding );
		_siteName = Ui::Text::String (min);
		_title = Ui::Text::String (min);
		_Beschreibung = Ui::Text::String (min);
		wenn (Faktencheck) {
			Faktencheck-> Fußzeile = Ui::Text::String (
				st::factcheckFooterStyle,
				tr::lng_factcheck_bottom (
					tr::jetzt,
					lt_Land,
					LookupFactcheckCountryName (
						LookupFactcheckCountryIso2 (Artikel))),
				kDefaultTextOptions ,
				min);
		}
	}
	const  auto lineHeight = UnitedLineHeight ();
	if (!_openl && (!_data-> url . isEmpty () || gesponsert || Faktencheck)) {
		const  auto original = _parent-> data ()-> originalText ();
		const  auto previewOfHiddenUrl = [&] {
			wenn (_data-> Typ == Webseitentyp::BotApp) {
				// Bot-Web-Apps zeigen bei versteckten URLs immer eine Bestätigung an.
				//
				// Aber von der dedizierten Schaltfläche "App öffnen" wollen wir nicht
				// um eine Benutzerbestätigung anzufordern, wenn die App nicht zum ersten Mal geöffnet wird.
				 gibt false zurück ;
			}
			const  auto vereinfachen = []( const QString &url) {
				automatisches Ergebnis = url.toLower ( );
				wenn (Ergebnis. endet mit ( ' / ' )) {
					Ergebnis. hacken ( 1 );
				}
				const  auto Präfixe = { u" http:// " _q, u" https:// " _q };
				für ( const  auto &prefix : Präfixe) {
					if (Ergebnis.startetMit ( Präfix )) {
						Ergebnis = Ergebnis.mitte (Präfix.größe ( ) );
						brechen ;
					}
				}
				Ergebnis zurückgeben ;
			};
			const  auto vereinfacht = vereinfachen (_data-> url );
			für ( const  auto &entity: original.entities ) {
				wenn (entity.type ( ) != EntityType::Url) {
					weitermachen ;
				}
				const  auto  link = original.text.mid (​​
					Entität.Offset ( ),
					Entität.Länge ( ));
				wenn ( vereinfachen ( Link ) == vereinfacht) {
					 gibt false zurück ;
				}
			}
			gibt true zurück;
		}();
		_openl = _data-> iv
			? IvClickHandler (_Daten, Original)
			: (Vorschau der versteckten URL || UrlClickHandler::IsSuspicious(
				_Daten->URL))
			? std::make_shared<HiddenUrlClickHandler>(_data-> url , _data-> url )
			: std::make_shared<UrlClickHandler>(_data-> url , true );
		wenn (_data->document
			&& (_data-> Dokument -> isWallPaper ()
				|| _data-> document -> isTheme ())) {
			_openl = std::make_shared<DocumentWrappedClickHandler>(
				std::move (_openl),
				_Daten->Dokument,
				_parent-> Daten ()-> fullId ());
		}
		wenn (_sponsoredData) {
			_openl = SponsoredLink (_sponsoredData-> hasExternalLink
				? _data-> URL
				: QString ());
		wenn (gesponsert) {
			_openl = SponsoredLink (_data-> url , gesponsert-> isLinkInternal );
			wenn (gesponsert->kannmelden) {
				sponsored->hint.link = ÜberSponsoredClickHandler();
			}
		} sonst wenn (Faktencheck) {
			const  auto item = _parent-> data ();
			const auto iso2 = LookupFactcheckCountryIso2(Element);
			wenn (!iso2.isEmpty()) {
				factcheck->hint.link = ÜberFactcheckClickHandler(iso2);
			}
		} anders {
			_openl = _data-> iv
				? IvClickHandler (_Daten, Original)
				: (Vorschau der versteckten URL || UrlClickHandler::IsSuspicious(
					_Daten->URL))
				? std::make_shared<HiddenUrlClickHandler>(_data-> url , _data-> url )
				: std::make_shared<UrlClickHandler>(_data-> url , true );
			wenn (_data->document
				&& (_data-> Dokument -> isWallPaper ()
					|| _data-> document -> isTheme ())) {
				_openl = std::make_shared<DocumentWrappedClickHandler>(
					std::move (_openl),
					_Daten->Dokument,
					_parent-> Daten ()-> fullId ());
			}
		}
	}

	// Layout initialisieren
	const  auto title = TextUtilities::SingleLine ( _data- > title.isEmpty ( ))
		? _data-> Autor
		: _data-> Titel );
	using Flag = MediaWebPageFlag;
	wenn (_data-> hasLargeMedia && (_flags & Flag::ForceLargeMedia)) {
		_asArticle = 0 ;
	} sonst  wenn (_data-> hasLargeMedia && (_flags & Flag::ForceSmallMedia)) {
		_asArticle = 1 ;
	} anders {
		_asArticle = _data-> computeDefaultSmallMedia ();
	}
	// Anfügen initialisieren
	wenn (!_attach && !_asArticle) {
		_attach = Anfügen erstellen (
			_Elternteil,
			_data-> Dokument ,
			_data-> Foto ,
				_Daten-> URL );
	}
	// Zeichenfolgen initieren
	wenn (_description.isEmpty () und !_data-> Beschreibung . Text .isEmpty ( ) ) {
		const  auto &text = _data-> Beschreibung ;
		wenn ( istLogEntryOriginal ()) {
			// Layout für kleine Blasen korrigieren
			// (Protokolleinträge zum Bearbeiten von Medienuntertiteln einschränken).
			_description = Ui::Text::String (st::minFotogröße
				- rechteck::m::sum::h (Auffüllung));
		}
		mit MarkedTextContext = Core::MarkedTextContext;
		automatischer Kontext = MarkedTextContext{
			. Sitzung = & Verlauf ()-> Sitzung (),
			. customEmojiRepaint = [=] { _parent-> customEmojiRepaintbenutzerdefiniertEmojiNeu streichen (); },
		};
		if (_data-> siteName == u" Twitter " _q) {
			Kontext.Typ = MarkedTextContext ::HashtagMentionType::Twitter;
		} sonst  wenn (_data-> siteName == u" Instagram " _q) {
			Kontext. Typ = MarkedTextContext::HashtagMentionType::Instagram;
		}
		_Beschreibung.setMarkedText (​
			st::Webseitenbeschreibungsstil,
			Text,
			Ui::WebpageTextDescriptionOptions (),
			Kontext);
	}
	const  auto siteName = _data-> angezeigterSiteName ();
	wenn (!siteName.isEmpty ( )) {
		_siteNameLines = 1 ;
		_siteName.setMarkedText (​
			st::WebseitenTitelStil,
			Ui::Text::Link (Sitename, _Daten-> URL ),
			Ui::WebpageTextTitleOptions ());
	}
	wenn (_title.isEmpty ( ) && !title.isEmpty ( )) {
		wenn (!_siteNameLines && !_data-> url . isEmpty ()) {
			_title.setMarkedText (​
				st::WebseitenTitelStil,Webseitentitelstil ,
				Ui::Text::LinkText ::Linktitel , _data-> (Titel, _Daten-> URL ),
				Ui::WebpageTextTitleOptions ());
		} anders {
			_title.setText (​
				st::WebseitenTitelStil,
				Titel,
				Ui::WebpageTextTitleOptions ());
		}
	}
	// Dimensionen initialisieren
	const  auto skipBlockWidth = _parent-> skipBlockWidthBlockbreite überspringen ();
	auto maxWidth = Blockbreite überspringen;
	Auto min. Höhe = 0 ;minHöhe =
	Konstante  auto siteNameHeight = _siteName.isEmpty () ? 0 : Zeilenhöhe ;_Sitename .
	Konstante  auto titleMinHeight = _title.isEmpty () ? 0 : Zeilenhöhe ;TitelMindesthöhe = _Titel.
	const  auto factcheckMetrics = Faktencheck
		? computeFactcheckMetrics (_description.minHeight ( ))
		: FactcheckMetrics ();
	const  auto descMaxLines = Faktencheck
		? factcheckMetrics. Zeilen
		: istLogEntryOriginal ()
		? kMaxOriginalEntryLines
		: ( 3 + (siteNameHeight ? 0 : 1 ) + (titleMinHeight ? 0 : 1 ));
	const  auto descriptionMinHeight = _description.isEmpty ( )
		? 0
		: std::min (_description.minHeight (), descMaxLines * Zeilenhöhe) ;
	const  auto articleMinHeight = siteNameHeight
		+ TitelMinHeightTitelMinHeight
		+ BeschreibungMinHeight;
	const  auto articlePhotoMaxWidth = _asArticle
		? st::webPagePhotoDelta
			+ std::max (
				ArticleThumbWidth (_data-> Foto , ArtikelMinHeight),
				Zeilenhöhe)
		: 0 ;
	wenn (!_siteName.isEmpty ( )) {
		acquire_max (maxBreite , _siteName.maxBreite () + ArtikelFotoMaxBreite);
		minHeight += Zeilenhöhe;
	}
	wenn (!_title.isEmpty ( )) {
		akkumuliere_max (maxBreite, _title.maxBreite ( ) + ArtikelFotoMaxBreite);
		minHeight += TitelMinHeight;
	}
	wenn (!_Beschreibung.istEmpty ( )) {
		akkumuliere_max (
			maxBreite,
			_description.maxWidth ( ) + ArtikelfotoMaxWidth);
		minHeight += BeschreibungMinHeight;
	}
	wenn (Faktencheck && Faktencheck-> erweitert ) {
		acquire_max (maxWidth, Faktencheck-> Fußzeile.maxWidth ( ) );
		minHeight += st::factcheckFooterSkip + factcheck-> footer . minHeight ();
	}
	wenn (_attach) {
		const  auto attachAtTop = _siteName.isEmpty ( )
			&& _title.isEmpty ( )
			&& _description.isEmpty ( );
		wenn (!attachAtTop) {
			minHeight += st::mediaInBubbleSkip;
		}
		_attach-> initDimensions ();
		const  auto bubble = _attach-> bubbleMargins ();
		auto maxMediaWidth = _attach-> maxWidth () - rect::m::sum::h (Blase);
		wenn ( isBubbleBottom () und _attach-> customInfoLayout ()) {
			maxMedienbreite += Breite des überspringenden Blocks;
		}
		acquire_max (max.Breite, max.Medienbreite);
		minHeight += _attach-> minHeight () - rect::m::sum::v (Blase);
	}
	wenn (_data-> Typ == Webseitentyp::Video && _data-> Dauer ) {
		_duration = Ui::FormatDurationText (_data-> Dauer );
		_durationWidth = st::msgDateFont-> Breite (_duration);
	}
	wenn (!_openButton.isEmpty ( )) {
		maxWidth += rechteck::m::sum::h (st::historyPageButtonPadding)
			+ _openButton.maxWidth ( );
	}
	maxWidth += rechteck::m::sum::h (Polsterung);
	minHeight += rechteck::m::sum::v (Polsterung);
	wenn (_asArticle) {
		minHeight = resizeGetHeight (maxBreite);
	}
	wenn ( const  auto hint = hintData ()) {
		Hinweis -> Breite davor = st::Webseitentitelstil.Schriftart - > Breite (Sitename);
		const  auto &font = st::webPageSponsoredHintFont;
		Hinweis-> Text = gesponsert
			? tr::lng_sponsored_message_revenue_button (tr::jetzt)
			: tr::lng_factcheck_whats_this (tr::jetzt);
		Hinweis-> Größe = QSize (
			Schriftart-> Breite (Hinweis-> Text ) + Schriftart-> Höhe ,
			Schriftart -> Höhe );
		maxWidth += Hinweis-> Größe . Breite ();
	}
	return { max.Breite, min.Höhe };
}
QSize WebPage::countCurrentSize ( int neueBreite) {
	wenn (_data-> pendingTill || _data-> fehlgeschlagen ) {
		return { neueBreite, min.Höhe () };
	}
	const  auto padding = inBubblePadding () + innerMargin ();
	const  auto innerWidth = neueWidth - rect::m::sum::h (Polsterung);
	auto neue Höhe = 0 ;
	const  auto stickerSet = stickerSetData ();
	const  auto factcheck = factcheckData ();
	const  auto specialRightPix = ( gesponserteData () || StickerSet);
	const  auto lineHeight = UnitedLineHeight ();
	const  auto factcheckMetrics = Faktencheck
		? computeFactcheckMetrics (_description.countHeight ( innereBreite))
		: FactcheckMetrics ();
	wenn (Faktencheck) {
		factcheck-> expandable = factcheckMetrics. expandable ? 1 : 0 ;
		factcheck-> erweitert = factcheckMetrics. erweitert ? 1 : 0 ;
		_openl = factcheck-> erweiterbar
			? ToggleFactcheckClickHandler (_übergeordnet)
			: nullptr ;
	}
	const  auto linesMax = Faktencheck
		? (factcheckMetrics. Zeilen + 1 )
		: (specialRightPix || istOriginalLogEntry ())
		? kMaxOriginalEntryLines
		: 5 ;
	const  auto siteNameHeight = _siteNameLines? Zeilenhöhe: 0 ;
	const  auto twoTitleLines = 2 * st::webPageTitleFont-> Höhe ;
	const  auto descriptionLineHeight = st::webPageDescriptionFont-> Höhe ;
	wenn ( alsArtikel () || speziellesRechtesBild) {
		constexpr  auto  kSponsoredUserpicLines = 2 ;
		_pixh = Zeilenhöhe
			* (Aufkleberset
				? kStickerSetLines
				: SpezialRechtsBilder
				? kSponsoredUserpicLines
				: ZeilenMax);
		Tun {
			_pixw = SpezialRechtesBild
				?_pixh
				: ArticleThumbWidth (_Daten-> Foto , _pixh);
			const  auto wleft = innere Breite
				- st::webPagePhotoDelta
				- std::max (_pixw, Zeilenhöhe);
			neueHöhe = SiteNameHeight;
			wenn (_title.isEmpty ( )) {
				_titleLines = 0 ;
			} anders {
				_titleLines = (_title.AnzahlHöhen ( bleft ) < zweiTitelzeilen)
					? 1
					: 2 ;
				neueHöhe += _titleLines * Zeilenhöhe;
			}
			const  auto descriptionHeight = _description.countHeight ( wleft );
			const  auto restLines = (linesMax - _siteNameLines - _titleLines);
			if (Beschreibungshöhe < Restzeilen * Beschreibungszeilenhöhe) {
				// Wir haben die Höhe für alle Zeilen.
				_descriptionLines = - 1 ;
				neueHöhe += Beschreibungshöhe;
			} anders {
				_descriptionLines = restLines;
				neueHöhe += _Beschreibungszeilen * Zeilenhöhe;
			}
			wenn (neueHöhe >= _pixh) {
				brechen ;
			}
			_pixh -= Zeilenhöhe;
		} während (_pixh > Zeilenhöhe);
	} anders {
		neueHöhe = SiteNameHeight;
		wenn (_title.isEmpty ( )) {
			_titleLines = 0 ;
		} anders {
			_titleLines = (_title. countHeight (innereBreite) < zweiTitelzeilen)
				? 1
				: 2 ;
			neueHöhe += _titleLines * Zeilenhöhe;
		}
		wenn (_description.isEmpty ( )) {
			_Beschreibungszeilen = 0 ;
		} anders {
			const  auto restLines = (linesMax - _siteNameLines - _titleLines);
			const  auto descriptionHeight = _description.countHeight (
				innereBreite);
			if (Beschreibungshöhe < Restzeilen * Beschreibungszeilenhöhe) {
				// Wir haben die Höhe für alle Zeilen.
				_descriptionLines = - 1 ;
				neueHöhe += Beschreibungshöhe;
			} anders {
				_descriptionLines = restLines;
				neueHöhe += _Beschreibungszeilen * Zeilenhöhe;
			}
		}
		wenn (Faktencheck && Faktencheck-> erweitert ) {
			factcheck-> footerHeight = st::factcheckFooterSkip
				+ factcheck-> Fußzeile .countHeight ( innereBreite);
			neueHöhe += Faktencheck-> FußzeilenHöhe ;
		}
		wenn (_attach) {
			const  auto attachAtTop = !_siteNameLines
				&& !_Titelzeilen
				&& !_Beschreibungszeilen;
			wenn (!attachAtTop) {
				neueHöhe += st::mediaInBubbleSkip;
			}
			const  auto bubble = _attach-> bubbleMargins ();
			_attach-> resizeGetHeight (innereBreite + rechteck::m::sum::h (Blase));
			neueHöhe += _attach-> Höhe () - Rechteck::m::Summe::v (Blase);
		}
	}
	neueHöhe += rechteck::m::sum::v (Polsterung);
	return { neueBreite, neueHöhe };
}
TextSelection WebPage::toTitleSelection (TextSelection-Auswahl) const {
	 gibt UnshiftItemSelection (Auswahl, _SiteName) zurück ;
}
TextSelection WebPage::fromTitleSelection (TextSelection-Auswahl) const {
	 gibt ShiftItemSelection (Auswahl, _SiteName) zurück ;
}
TextSelection WebPage::toDescriptionSelection (TextSelection-Auswahl) const {
	returniere  UnshiftItemSelection ( toTitleSelection (Auswahl), _title);
}
Textauswahl Webseite::fromDescriptionSelection (
		TextSelection Auswahl) const {
	returniere  ShiftItemSelection ( fromTitleSelection (Auswahl), _Titel);
}
void  Webseite::refreshParentId (not_null<HistoryItem*> realParent) {
	wenn (_attach) {
		_attach-> refreshParentId (echtesParent);
	}
}
void  Webseite::ensurePhotoMediaCreated () const {
	Erwartet (_data-> photo != nullptr );
	wenn (_photoMedia) {
		zurückkehren ;
	}
	_photoMedia = _data-> Foto -> createMediaView ();
	const  auto contextId = _parent-> data ()-> fullId ();
	_photoMedia-> gesucht (Data::PhotoSize::Thumbnail, contextId);
	Verlauf () -> Eigentümer ().registerHeavyViewPart ( _parent);
}
bool  Webseite::hasHeavyPart () const {
	wenn ( const  auto stickerSet = stickerSetData ()) {
		für ( const  auto &part: stickerSet-> Ansichten ) {
			wenn (Teil-> hatschweresTeil ()) {
				 gibt true zurück ;
			}
		}
	}
	return _photoMedia
		|| (_attach ? _attach-> hasHeavyPart () : false );
}
Leere  Webseite::unloadHeavyPart () {
	wenn (_attach) {
		_anhängen-> SchweresTeil entladen ();
	}
	_Beschreibung.unloadPersistentAnimation ( );
	_photoMedia = nullptr ;
	wenn ( const  auto stickerSet = stickerSetData ()) {
		für ( const  auto &part: stickerSet-> Ansichten ) {
			Teil-> SchweresTeil entladen ();
		}
	}
}
void  WebPage::draw (Painter &p, const PaintContext &context) const {
	wenn ( Breite () < rechteck::m::sum::h (st::msgPadding) + 1 ) {
		zurückkehren ;
	}
	const  auto st = Kontext.st ;
	const  auto sti = Kontext.Bildstil ( );
	const  auto stm = Kontext.Nachrichtenstil ( );
	const  auto bubble = _attach ? _attach-> bubbleMargins () : QMargins ();
	const  auto full = Rechteck ( aktuelle Größe ());
	const  auto outer = voll - inBubblePadding ();
	const  auto inner = outer - innerMargin ();
	const  auto attachAdditionalInfoText = _attach
		?_attach-> zusätzlicherInfoString ()
		: QString ();
	auto tshift = inner.top ( );
	auto paintw = innen.breite ( );
	const  auto gesponsert = gesponserteDaten ();
	const  auto factcheck = factcheckData ();
	const  auto selected = Kontext.ausgewählt ( );
	const  auto view = übergeordnetes Element ();
	const  auto from = Ansicht -> Daten () -> contentColorsFrom ();
	const  auto colorIndex = Faktencheck
		? 0  // rot
		: (gesponsert && gesponsert-> Farbindex )
		? gesponsert-> Farbindex
		: aus
		? von-> Farbindex ()
		: Ansicht-> Farbindex ();
	const  auto cache = Kontext.outbg
		? stm-> replyCache [st-> colorPatternIndex (colorIndex)]. get ()
		: st-> coloredReplyCache (ausgewählt, Farbindex).get ( );
	const  auto backgroundEmojiId = Faktencheck
		? Dokument-ID ()
		: (gesponsert und gesponsert-> backgroundEmojiId )
		? gesponsert-> HintergrundEmojiId
		: aus
		? von-> backgroundEmojiId ()
		: Dokument-ID ();
	const  auto backgroundEmoji = backgroundEmojiId
		?st-> backgroundEmojiData (backgroundEmojiId).get ( )
		: nullptr ;
	const  auto backgroundEmojiCache = HintergrundEmoji
		? &backgroundEmoji-> Caches [ Ui::BackgroundEmojiData::CacheIndex (
			ausgewählt,
			Kontext.outbg​ ,
			WAHR ,
			Farbindex + 1 )]
		: nullptr ;
	Ui::Text::ValidateQuotePaintCache (*cache, _st);
	Ui::Text::FillQuotePaint (p, äußerer, *Cache, _st);
	Wenn (Hintergrund-Emoji) {
		ValidierenHintergrundEmoji (
			HintergrundEmojiId,
			HintergrundEmoji,
			HintergrundEmojiCache,
			Zwischenspeicher,
			Sicht);
		wenn (!backgroundEmojiCache-> frames [ 0 ]. isNull ()) {
			FillBackgroundEmoji (p, außen, falsch , *backgroundEmojiCache);
		}
	} sonst  wenn (factcheck && factcheck-> erweiterbar ) {
		const  auto &icon = factcheck-> erweitert ? _st.collapse : _st.expand ;
		const  auto &position = factcheck-> erweitert
? 			_st.collapsePosition
: 			_st.expandPosition:
		Symbol. Farbe (
			P,
			außen. x () + außen. breite () - symbol. breite () - position. x (),
			außen.y ( ) + außen.höhe ( ) - Symbol.höhe () - Position.y ( ) ,
			Breite ());
	}
	wenn (_ripple) {
		_ripple-> paint (p, außen.x (), außen.y ( ), Breite ( ), &cache-> bg );
		wenn (_ripple-> leer ()) {
			_ripple = nullptr ;
		}
	}
	auto Linienhöhe = UnitedLineHeight ();
	wenn ( const  auto stickerSet = stickerSetData ()) {
		const  auto viewsCount = stickerSet-> Ansichten.Größe ( ) ;
		const  auto box = _pixh;
		const  auto topLeft = QPoint (inner.left ( ) + paintw-box, tshift);
		const  auto side = std::ceil ( std::sqrt (Anzahl der Ansichten));
		const  auto single = Box / Seite;
		für ( auto i = 0 ; i < Seite; i++) {
			für ( auto j = 0 ; j < Seite; j++) {
				const  auto  index = i * Seite + j;
				wenn (Anzahl der Aufrufe <= Index ) {
					brechen ;
				}
				const  auto &view = stickerSet-> Ansichten [ Index ];
				const  auto size = Ansicht -> AnzahlOptimalGröße ();
				const  auto offsetX = (single - Größe.Breite ( )) / 2 .;
				const  auto offsetY = (single - size.height ( )) / 2 .;
				const  auto x = j * single + offsetX;
				const  auto y = i * single + offsetY;
				Ansicht -> Zeichnen (p, Kontext, QRect ( QPoint (x, y) + topLeft, Größe));
			}
		}
		malen -= Kasten;
	} sonst  wenn ( alsArtikel ()) {
		sicherstellen, dass PhotoMediaCreated ();
		auto pix = QPixmap ();
		const  auto pw = qMax (_pixw, Zeilenhöhe);
		const  auto ph = _pixh;
		auto pixw = _pixw;
		auto pixh = ArticleThumbHeight (_photoMedia.get ( ), _pixw);
		const  auto maxsize = _photoMedia-> Größe (Daten::Fotogröße::Thumbnail);
		const  auto maxw = style::Scale konvertieren ( maxsize.width ());
		const  auto maxh = style::Skalierung konvertieren ( maxsize.höhe ());
		wenn (pixw * ph != pixh * pw) {
			const  auto coef = (pixw * ph > pixh * pw)
				? std::min (ph / float64 (pixh), maxh / float64 (pixh))
				: std::min (pw / float64 (pixw), maxw / float64 (pixw));
			pixh = std::round (pixh * koef);
			pixw = std::round (pixw * koef);
		}
		const  auto size = QSize (pixw, pixh);
		Konstante  auto args = Bilder::PrepareArgs{
			. Optionen = Bilder::Option::RoundSmall,
			. äußere = { pw, ph },
		};
		 Namespace-  Daten verwenden ;
		wenn ( const  auto thumbnail = _photoMedia-> Bild (PhotoSize::Thumbnail)) {
			pix = Miniaturansicht -> pixSingle (Größe, Argumente);
		} sonst  wenn ( const  auto small = _photoMedia-> image (PhotoSize::Small)) {
			pix = klein -> pixSingle (Größe, Argumente unscharf ());
		} sonst  wenn ( const  auto blurred = _photoMedia-> thumbnailInline ()) {
			pix = unscharf -> pixSingle (Größe, Argumente unscharf ());
		}
		p.drawPixmapLeft (innen.links ( ) + paintw - pw, tshift, width (), pix);
		wenn (Kontext.ausgewählt ( )) {
			const  auto st = Kontext.st ;
			Ui::FillRoundRect (
				P,
				Stil::rtlrect (
					innen. links () + paintw - pw,
					tverschiebung,
					pw,
					_pixh,
					Breite ()),
				st-> msgSelectOverlay (),
				st-> msgSelectOverlayCorners (Ui::CachedCornerRadius::Small));
		}
		wenn (!gesponsert) {
			// Fotobreite in gesponserten Nachrichten ignorieren,
			// da seine Breite nur den Titel beeinflusst.
			paintw -= pw + st::webPagePhotoDelta;
		}
	}
	wenn (_siteNameLines) {
		p. setPen (Cache-> Symbol );
		p.setTextPalette ( Kontext.outbg​
			? stm-> halbfette Palette
			: st-> farbigeTextPalette (ausgewählt, Farbindex));
		const  auto endskip = _siteName.hasSkipBlock ( )
			?_parent-> Blockbreite überspringen ()
			: 0 ;
		_siteName.drawLeftElided (​
			P,
			innen. links (),
			tverschiebung,
			malen,
			Breite (),
			_siteNameLines,
			Stil::al_left,
			0 ,
			- 1 ,
			Ende überspringen,
			FALSCH ,
			Kontext. Auswahl );
		const  auto hint = hintData ();
		if (Hinweis && (paintw > Hinweis-> BreiteVorher + Hinweis-> Größe . Breite ())) {
			automatische Farbe = Cache-> Symbol ;
			Farbe.setAlphaF (Farbe.alphaF () * 0,15 ) ;
			const  auto Höhe = st::webPageSponsoredHintFont-> Höhe ;
			const  auto Radius = Höhe / 2 ;
			Hinweis-> letztePosition = QPointF (
Radius + 				innerer.links () + Hinweis-> BreiteVorher ,
				tshift + (_siteName.style ( ) -> Schriftart -> Höhe - Höhe) / 2 .);
			if (Hinweis-> Welligkeit ) {
				Hinweis-> Wellen- > Malen (
					P,
					Hinweis-> letztePosition . x (),
Hinweis- 					> letztePosition.y ( ),
					Breite (),
					&cache-> bg );
				wenn (Hinweis-> Welle- > leer ()) {
					Hinweis -> Welligkeit = nullptr ;
				}
			}
			const  auto rect = QRectF (Hinweis-> letztePosition , Hinweis-> Größe );
			auto hq = PainterHighQualityEnabler (p);
			p. setPen (Qt::NoPen);
			p. setBrush (Farbe);
			p. drawRoundedRect (Rechteck, Radius, Radius);
			p. setPen (Cache-> Symbol );
			p. setBrush (Qt::NoBrush);
			p. setFont (st::webPageSponsoredHintFont);
			p. drawText (rechteck, hint-> text , style::al_center);
		}
		tshift += Zeilenhöhe;
		P. setTextPalette (stm-> textPalette );
	}
	p. setPen (stm-> historyTextFg );
	wenn (_titleLines) {
		const  auto endskip = _title.hasSkipBlock ( )
			?_parent-> Blockbreite überspringen ()
			: 0 ;
		const  auto titleWidth = gesponsert
			? (paintw – _pixh – st::webPagePhotoDelta)
			: malenw;
		_title.drawLeftElided (​
			P,
			innen. links (),
			tverschiebung,
			Titelbreite,
			Breite (),
			_Titelzeilen,
			Stil::al_left,
			0 ,
			- 1 ,
			Ende überspringen,
			FALSCH ,
			toTitleSelection (Kontext.Auswahl ) );
		tshift += _titleLines * Zeilenhöhe;
	}
	wenn (_Beschreibungszeilen) {
		const  auto endskip = _description.hasSkipBlock ( )
			?_parent-> Blockbreite überspringen ()
			: 0 ;
		_parent-> prepareCustomEmojiPaint (p, Kontext, _Beschreibung);
		_Beschreibung.zeichnen ( p, {
			. position = { inner. left (), tshift },
			. äußereBreite = Breite (),
			. verfügbareBreite = paintw,
			. spoiler = Ui::Text::DefaultSpoilerCache (),
			. jetzt = Kontext. jetzt ,
			. pausedEmoji = Kontext. pausiert || Ein (Energiesparen:: kEmojiChat ),
			. pausedSpoiler = Kontext. pausiert || Ein (Energiesparen:: kChatSpoiler ),
			. Auswahl = toDescriptionSelection (Kontext. Auswahl ),
			.elisionHeight = ((_descriptionLines > 0 )
				? (_Beschreibungszeilen * Zeilenhöhe)
				: 0 ),
			. elisionRemoveFromEnd = (_descriptionLines > 0 ) ? endskip : 0 ,
		});
		tshift += (_Beschreibungszeilen > 0 )
			? (_Beschreibungszeilen * Zeilenhöhe)
			: _Beschreibung.AnzahlHöhe ( Farbe);
	}
	wenn (Faktencheck && Faktencheck-> erweitert ) {
		const  auto skip = st::factcheckFooterSkip;
		const  auto line = st::Zeilenbreite;
		const  auto separatorTop = tshift + skip / 2 ;
		automatische Farbe = Cache-> Symbol ;
		Farbe.setAlphaF (Farbe.alphaF () * 0,3 ) ;
		p.fillRect ( innen.links (), Trennzeichen oben , Farbe, Linie, Farbe);
		p. setPen (Cache-> Symbol );
		Faktencheck-> Fußzeile . zeichnen (p, {
			. position = { inner. left (), tshift + skip },
			. äußereBreite = Breite (),
			. verfügbareBreite = paintw,
		});
		tshift += Faktencheck-> Fußzeilenhöhe ;
	}
	wenn (_attach) {
		const  auto attachAtTop = !_siteNameLines
			&& !_Titelzeilen
			&& !_Beschreibungszeilen;
		wenn (!attachAtTop) {
			tshift += st::mediaInBubbleSkip;
		}
		const  auto attachLeft = rtl ()
			? ( Breite () – (innen.links ( ) – Blase.links ( )) – _attach-> Breite ())
			: (innen.links ( ) - Blase.links ( ));
		const  auto attachTop = tshift - Blase.oben ( );
		p. übersetzen (links anhängen, oben anhängen);
		_attach-> zeichnen (p, Kontext. übersetzt (
			-Links anhängen,
			-AnhängenOben
		).mitAuswahl ( Kontext.ausgewählt ( )
			? Volle Auswahl
			: Textauswahl ()));
		const  auto pixwidth = _attach-> Breite ();
		const  auto pixheight = _attach-> Höhe ();
		wenn (_data-> Typ == Webseitentyp::Video
&& 			_collage.leer ()
			&& _data-> Foto
			&& !_data-> Dokument ) {
			wenn (_attach-> isReadyForOpen ()) {
				wenn (_data-> siteName == u" YouTube " _q) {
					st-> youtubeIcon () .farbe (
						P,
						(Pixelbreite - st::youtubeIcon.Breite ( )) / 2 ,
						(Pixelhöhe - st::youtubeIcon. Höhe ()) / 2 ,
						Breite ());
				} anders {
					st-> videoIcon () .farbe (
						P,
						(Pixelbreite - st::videoIcon.Breite ( )) / 2 ,
						(Pixelhöhe - st::videoIcon. Höhe ()) / 2 ,
						Breite ());
				}
			}
			wenn (_DauerBreite) {
				const  auto dateX = Pixelbreite
					- _DauerBreite
					- st::msgDateImgDelta
					- 2 * st::msgDateImgPadding.x ( );
				const  auto dateY = Pixelhöhe
					- st::msgDateFont-> Höhe
					- 2 * st::msgDateImgPadding.y ( )
					- st::msgDateImgDelta;
				const  auto dateW = Pixelbreite - dateX - st::msgDateImgDelta;
				const  auto dateH = Pixelhöhe - dateY - st::msgDateImgDelta;
				Ui::FillRoundRect (
					P,
					DatumX,
					DatumJ,
					DatumW,
					DatumH,
					sti-> msgDateImgBg ,
					sti-> msgDateImgBgCorners );
				p. setFont (st::msgDateFont);
				p.setPen ( st-> msgDateImgFg ());
				S. zeichneTextLeft (
					dateX + st::msgDateImgPadding. x (),
					dateY + st::msgDateImgPadding. y (),
					Pixelbreite,
					_Dauer);
			}
		}
		p. übersetzen (-attachLeft, -attachTop);
		wenn (!attachAdditionalInfoText.isEmpty ( )) {
			p. setFont (st::msgDateFont);
			p.setPen (stm-> msgDateFg ) ;
			S. zeichneTextLeft (
				st::msgPadding.left ( ),
				außen.y ( ) + außen.höhe ( ) + st::mediaInBubbleSkip,
				Breite (),
				addAdditionalInfoText);
		}
	}
	wenn (!_openButton.isEmpty ( )) {
		p. setFont (st::halbfetteSchriftart);
		p. setPen (Cache-> Symbol );
		const  auto end = inner.y ( ) + inner.height ( ) + _st.padding.bottom ( ) ;
		const  auto line = st::historyPageButtonLine;
		automatische Farbe = Cache-> Symbol ;
		Farbe.setAlphaF (Farbe.alphaF () * 0,3 ) ;
		p.fillRect (inneres.x ( ), Ende, innere.breite (), Linie , Farbe) ;
		_openButton.zeichnen ( p, {
			. Position = QPoint (
				inner.x ( ) + (inner.breite ( ) - _openButton.maxWidth ( )) / 2 ,
				Ende + st::historyPageButtonPadding. oben ()),
			. verfügbareBreite = paintw,
			. jetzt = Kontext. jetzt ,
		});
	}
}
bool  Webseite::asArticle () const {
	return _asArticle && (_data-> Foto != nullptr );
}
Webseite::StickerSetData * Webseite::stickerSetData () const {
	return std::get_if<StickerSetData>(_additionalData.get ( ));
}
Webseite::GesponserteDaten * Webseite::GesponserteDaten () const {
	return std:: get_if <SponsoredData>(_additionalData.get ));
}
Webseite::FactcheckData * Webseite::factcheckData () const {
	return std::get_if<FactcheckData>( _additionalData.get ));
}
Webseite::HinweisDaten * Webseite::HinweisDaten () const {
	wenn ( const  auto gesponsert = gesponserteDaten ()) {
		gesponserten -> Hinweis zurückgeben . Link ? &gesponsert -> Hinweis : nullptr ;
	} sonst  wenn ( const  auto factcheck = factcheckData ()) {
		returniere factcheck-> Hinweis . Link ? &factcheck-> Hinweis : nullptr ;
	}
	return  nullptr ;
}
TextState WebPage::textState (QPoint-Punkt, StateRequest-Anfrage) const {
	auto result = TextState (_parent);
	wenn ( Breite () < rechteck::m::sum::h (st::msgPadding) + 1 ) {
		zurückkehrenErgebnis 
	}
	const  auto gesponsert = gesponserteDaten ();
	const  auto bubble = _attach ? _attach-> bubbleMargins () : QMargins ();
	const  auto full = Rechteck ( aktuelle Größe ());
	auto outer = voll - inBubblePadding ();
	Wenn (gesponsert) {
		äußere. übersetzen ( 0 , st::msgDateFont-> Höhe ;
	}
	const  auto inner = outer - innerMargin ();
	auto tshift = inner.top ( );
	auto paintw = innen.breite ( );
	const  auto lineHeight = UnitedLineHeight ();
	auto inThumb = false ;
	wenn ( alsArtikel ()) {
		const  auto pw = std::max (_pixw, Zeilenhöhe);
		inThumb = Stil::rtlrect (
innen 			. links () + paintw - pw,
			tverschiebung,
			pw,
			_pixh,
			Breite ()). enthält (Punkt);
		paintw -= pw + st::webPagePhotoDelta;
	}
	auto symbolAdd = int ( 0 );
	wenn (_siteNameLines) {
		wenn (Punkt.y ( ) >= tshift && Punkt.y ( ) < tshift + Zeilenhöhe) {
			auto siteNameRequest = Ui::Text::StateRequestElided (
				Anfrage. fürText ));
			siteNameRequest. Zeilen _siteNameLines;
			Ergebnis = TextState (
				_Elternteil,
				_siteName.getStateElidedLeft​ (
					Punkt - QPoint (innen. links ), tshift),
					malen,
					Breite (),
					siteNameRequest));
		} sonst  wenn (Punkt. y ) >= tshift + Zeilenhöhe) {
			symbolAdd += _siteName. Länge );
		}
		tshift += Zeilenhöhe;
	}
	wenn (_titleLines) {
		wenn (Punkt. y () >= tshift
			&& Punkt.y () < tshift + _titleLines * Zeilenhöhe ) {
			automatische Titelanfrage = Ui::Text::StateRequestElided (
				Anfrage. fürText ));
			TitelAnfrage. Zeilen = _titleLines;
			Ergebnis = TextState (
				_Elternteil,
				_title.getStateElidedLeft​ (
					Punkt - QPoint (innen. links ), tshift),
					malen,
					Breite (),
					Titelanfrage));
		} sonst  wenn (Punkt.y ( ) >= tshift + _titleLines * lineHeight) {
			symbolAdd += _title.länge ( );
		}
		tshift += _titleLines * Zeilenhöhe;
	}
	wenn (_Beschreibungszeilen) {
		const  auto descriptionHeight = (_descriptionLines > 0 )
			? _descriptionLines * Zeilenhöhe
			: _Beschreibung.AnzahlHöhe ( Farbe);
		wenn (Punkt.y ( ) >= tshift && Punkt.y ( ) < tshift + Beschreibungshöhe) {
			wenn (_descriptionLines > 0 ) {
				automatische Beschreibungsanforderung = Ui::Text::StateRequestElided (
					Anfrage.fürText ( ));
				BeschreibungAnfrage.Zeilen = _BeschreibungZeilen;
				Ergebnis = TextState (
					_Elternteil,
					_Beschreibung.getStateElidedLeft (​
						Punkt - QPoint (innen.links ( ), tshift),
						malen,
						Breite (),
						BeschreibungAnfrage));
			} anders {
				Ergebnis = TextState (
					_Elternteil,
					_Beschreibung.getStateLeft (​
						Punkt - QPoint (innen.links ( ), tshift),
						malen,
						Breite (),
						Anfrage.fürText ( ) ));
			}
		} sonst  wenn (Punkt.y ( ) >= tshift + Beschreibungshöhe) {
			symbolAdd += _description.Länge ( );
		}
		tshift += Beschreibungshöhe;
	}
	wenn (inThumb) {
		Ergebnis. Link = _openl;
	} sonst  wenn (_attach) {
		const  auto attachAtTop = !_siteNameLines
			&& !_Titelzeilen
			&& !_Beschreibungszeilen;
		wenn (!attachAtTop) {
			tshift += st::mediaInBubbleSkip;
		}
		const  auto rect = QRect (
			innen. links (),
			tverschiebung,
			malen,
			inner.oben () + inner.höhe ( ) - tshift);
		if (Rechteck enthält (Punkt)) {
			const  auto attachLeft = rtl ()
				? Breite () – (innen.links ( ) – Blase.links ( )) – _attach-> Breite ()
				: (innen.links ( ) - Blase.links ( ));
			const  auto attachTop = tshift - Blase.oben ( );
			Ergebnis = _attach-> Textstatus (
				Punkt - QPoint (links anhängen, oben anhängen),
				Anfrage);
			wenn (Ergebnis. Cursor == CursorState::Vergrößern) {
				Ergebnis. Cursor = CursorState::None;
			} anders {
				Ergebnis.Link = replaceAttachLink ( Ergebnis.Link ) ;
			}
		}
	}
	wenn ((!Ergebnis.Link || gesponsert ) && äußere.enthält ( Punkt )) {
		Ergebnis. Link = _openl;
	}
	wenn ( const  auto hint = hintData ()) {
		const  auto check = Punkt
			– QPoint ( 0 , gesponsert? st::msgDateFont-> Höhe : 0 );
		const  auto hintRect = QRectF (Hinweis-> letztePosition , Hinweis-> Größe );
		if (hintRect. contains (check)) {
			Ergebnis. Link = Hinweis-> Link ;
		}
	}
	_lastPoint = Punkt - äußerer.topLeft ( );
	Ergebnis. Symbol += SymbolAdd;
	Ergebnis zurückgeben ;
}
ClickHandlerPtr Webseite::replaceAttachLink (
		const ClickHandlerPtr &link) const {
	wenn (!_attach-> isReadyForOpen ()
		|| (_siteName.istEmpty ( )
			&& _title.isEmpty ( )
			&& _Beschreibung.isEmpty ( ))
		|| (_data-> Dokument
			&& !_data-> Dokument -> isWallPaper ()
			&& !_data->document->isTheme())
		|| !_data->collage.items.empty()) {
		return link;
	}
	return _openl;
}
TextSelection WebPage::adjustSelection(
		TextSelection selection,
		TextSelectType type) const {
	if ((!_titleLines && !_descriptionLines)
	|| Auswahl. bis <= _siteName. Länge ()) {
		returniere _siteName.adjustSelection ( Auswahl , Typ);
	}
	const  auto titleLength = _siteName.länge () + _title.länge ( );
	const  auto titleSelection = _title.adjustSelection (
		toTitleSelection (Auswahl),
		Typ);
	wenn ((!_siteNameLines && !_descriptionLines)
		|| (Auswahl. von >= _siteName. Länge ()
			&& Auswahl. bis <= TitelLänge)) {
		Rückgabe  von Titelauswahl (Titelauswahl);
	}
	const  auto descriptionSelection = _description.adjustSelection (
		toDescriptionSelection (Auswahl),
		Typ);
	if ((!_siteNameLines && !_titleLines) || Auswahl. von >= titlesLength) {
		Rückgabe  von Beschreibungsauswahl (Beschreibungsauswahl);
	}
	zurückkehren {
		_siteName.adjustSelection ( Auswahl , Typ) .von ,
		(!_descriptionLines || Auswahl. bis <= Titellänge)
			? vonTitelAuswahl (TitelAuswahl). bis
			: vonBeschreibungsauswahl (Beschreibungsauswahl). bis ,
	};
}
uint16 Webseite::fullSelectionLength () const {
	returniere _siteName.Länge ( ) + _title.Länge ( ) + _description.Länge ( );
}
void  Webseite::clickHandlerActiveChanged (
		const ClickHandlerPtr &p,
		bool aktiv) {
	wenn (_attach) {
		_attach-> clickHandlerActiveChanged (p, aktiv);
	}
}
void  Webseite::clickHandlerPressedChanged (
		const ClickHandlerPtr &p,
		bool gedrückt) {
	const  auto hint = hintData ();
	if (Hinweis && Hinweis-> Link == p) {
		if (gedrückt) {
			wenn (!Hinweis-> Welligkeit ) {
				const  auto owner = & übergeordnetes Element ()-> Verlauf ()-> Eigentümer ();
				Hinweis -> Ripple = std::make_unique<Ui::RippleAnimation>(
					st::defaultRippleAnimation,
					Ui::RippleAnimation::RoundRectMask (
						Hinweis-> Größe ,
						_st. Radius ),
					[=] { Eigentümer-> requestViewRepaint ( übergeordnetes Element ()); });
			}
			const  auto full = Rechteck ( aktuelle Größe ());
			const  auto outer = voll - inBubblePadding ();
			Hinweis-> Ripple- > hinzufügen (_lastPoint
				+ außen.obenLinks ( )
				- Hinweis-> letztePosition.toPoint ( ) );
		} sonst  wenn (Hinweis-> Welligkeit ) {
			Hinweis -> Welligkeit -> letzter Stopp ();
		}
		zurückkehren ;
	}
	wenn (p == _openl) {
		if (gedrückt) {
			wenn (!_ripple) {
				const  auto full = Rechteck ( aktuelle Größe ());
				const  auto outer = voll - inBubblePadding ();
				const  auto owner = & übergeordnetes Element ()-> Verlauf ()-> Eigentümer ();
				_ripple = std::make_unique<Ui::RippleAnimation>(
					st::defaultRippleAnimation,
					Ui::RippleAnimation::RoundRectMask (
						äußere.Größe ( ),
						_st. Radius ),
					[=] { Eigentümer-> requestViewRepaint ( übergeordnetes Element ()); });
			}
			_ripple-> hinzufügen (_letzterPunkt);
		} sonst  wenn (_ripple) {
			_ripple-> letzterStopp ();
		}
	}
	wenn (_attach) {
		_attach-> clickHandlerPressedChanged (p, gedrückt);
	}
}
bool  Webseite::enforceBubbleWidth () const {
	Rückgabe (_attach != nullptr )
		&& (_data-> Dokument != nullptr )
		&& (_data-> document -> isWallPaper () || _data-> document -> isTheme ());
}
void  WebPage::playAnimation ( bool autoplay) {
	wenn (_attach) {
		wenn (Autoplay) {
			_attach-> autoplayAnimation ();
		} anders {
			_attach-> Animation abspielen ();
		}
	}
}
bool  Webseite::isDisplayed () const {
	return !_data-> ausstehendBis
		&& !_data-> fehlgeschlagen
		&& !_parent-> Daten ()-> Hat <HistoryMessageLogEntryOriginal>();
}
QString WebPage::additionalInfoString () const {
	gibt _attach zurück ? _attach-> additionalInfoString () : QString ();
}
bool  Webseite::toggleSelectionByHandlerClick ( const ClickHandlerPtr &p) const {
	returniere _attach und _attach-> toggleSelectionByHandlerClick (p);
}
bool  Webseite::allowTextSelectionByHandler ( const ClickHandlerPtr &p) const {
	Rückgabe (p == _openl);
}
bool  WebPage::dragItemByHandler ( const ClickHandlerPtr &p) const {
	return _attach && _attach-> dragItemByHandler (p);
}
TextForMimeData WebPage::selectedText (TextSelection-Auswahl) const {
	auto siteNameResult = _siteName.toTextForMimeData ( Auswahl );
	auto titleResult = _title.toTextForMimeData ( toTitleSelection ( Auswahl));
	auto descriptionResult = _description.toTextForMimeData (
		toDescriptionSelection (Auswahl));
	if (titleResult. empty () && descriptionResult. empty ()) {
		siteNameResult zurückgeben ;
	} sonst  wenn (siteNameResult.leer ( ) && descriptionResult.leer ( )) {
		Titelergebnis zurückgeben ;
	} else  if (siteNameResult. empty () && titleResult. empty ()) {
		gibt Beschreibungsergebnis zurück ;
	} sonst  wenn (siteNameResult. empty ()) {
		returniere Titelergebnis.anhängen ( ' \n ' ).anhängen ( std :: move (Beschreibungsergebnis));
	} else  if (titleResult. empty ()) {
		siteNameResult zurückgeben
			. anhängen ( ' \n ' )
			. anhängen ( std::move (Beschreibungsergebnis));
	} sonst  wenn (Beschreibungsergebnis.leer ( )) {
		Gibt siteNameResult zurück . anhängen ( ' \n ' ). append ( std::move (titleResult));
	}
	siteNameResult zurückgeben
		. anhängen ( ' \n ' )
		. append ( std::move (titleResult))
		. anhängen ( ' \n ' )
		. anhängen ( std::move (Beschreibungsergebnis));
}
QMargins WebPage::inBubblePadding () const {
	zurückkehren {
		st::msgPadding.left ( ),
		istBubbleTop () ? st::msgPadding.left ( ) : 0 ,
		st::msgPadding.right ( ),
		istBubbleBottom () ? (st::msgPadding.left ( ) + bottomInfoPadding ()) : 0
	};
}
QMargins WebPage::innerMargin () const {
	const  auto button = _openButton.isEmpty()
		? 0
		: st::historyPageButtonHeight;
	return _st. padding + QMargins ( 0 , 0 , 0 , Schaltfläche);
}
bool  Webseite::isLogEntryOriginal () const {
	return _parent->data()->isAdminLogEntry() && _parent->media() != this;
}
WebPage::FactcheckMetrics WebPage::computeFactcheckMetrics(
		int fullHeight) const {
	const auto possible = fullHeight / st::normalFont->height;
	//const auto expandable = (possible > kFactcheckCollapsedLines + 1);
	// Now always expandable because of the footer.	const auto expandable = true;
	const auto check = _parent->Get<Factcheck>();
	const auto expanded = check && check->expanded;
	const auto allowExpanding = (expanded || !expandable);
	return {
		.lines = allowExpanding ? possible : kFactcheckCollapsedLines,
		.expandable = expandable,
		.expanded = expanded,
	};
}
int WebPage::bottomInfoPadding() const {
	if (!isBubbleBottom()) {
		return 0;
	}
	auto result = st::msgDateFont->height;
	// We use padding greater than st::msgPadding.bottom() in the
	// bottom of the bubble so that the left line looks pretty.
	// but if we have bottom skip because of the info display
	// we don't need that additional padding so we replace it
	// back with st::msgPadding.bottom() instead of left().
	result += st::msgPadding.bottom() - st::msgPadding.left();
	return result;
}
WebPage::~WebPage() {
	history()->owner().unregisterWebPageView(_data, _parent);
	if (_photoMedia) {
		history()->owner().keepAlive(base::take(_photoMedia));
		_parent->checkHeavyPart();
	}
}
} // namespace HistoryView
