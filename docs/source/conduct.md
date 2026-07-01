"""
Animation Manim : Les 10 étapes de la glycolyse
Format vertical (1080x1920) pour Reels/TikTok/Shorts

Installation :
    pip install manim

Rendu (une phase à la fois, en qualité rapide pour prévisualiser) :
    manim -pql glycolyse.py Intro
    manim -pql glycolyse.py Phase1Investissement
    manim -pql glycolyse.py Phase2Production
    manim -pql glycolyse.py Bilan

Rendu en haute qualité (plus long) :
    manim -pqh glycolyse.py Intro Phase1Investissement Phase2Production Bilan

Note : pour concaténer les 4 vidéos en une seule, utilise ensuite CapCut
ou ffmpeg (ffmpeg -f concat -i liste.txt -c copy glycolyse_full.mp4)
"""

from manim import *

# ----- Configuration du format vertical (Reels/TikTok) -----
config.frame_width = 9
config.frame_height = 16
config.pixel_width = 1080
config.pixel_height = 1920

# ----- Palette de couleurs -----
COLOR_MOLECULE = "#4A90D9"   # bleu : glucose et dérivés
COLOR_ATP = "#F5C542"        # jaune : ATP
COLOR_NADH = "#E8934A"       # orange : NADH
COLOR_ENZYME = "#9B9B9B"     # gris : enzymes
COLOR_PYRUVATE = "#5FBF6F"   # vert : pyruvate final


def molecule_box(name: str, carbons: str = "") -> VGroup:
    """Crée une représentation simplifiée d'une molécule : un rectangle
    coloré avec son nom, et éventuellement le nombre de carbones."""
    box = RoundedRectangle(
        width=3.2, height=1.1, corner_radius=0.2,
        color=COLOR_MOLECULE, fill_opacity=0.25, stroke_width=3
    )
    label = Text(name, font_size=28, color=WHITE)
    label.move_to(box.get_center())
    group = VGroup(box, label)
    if carbons:
        sub = Text(carbons, font_size=20, color=GRAY_B)
        sub.next_to(box, DOWN, buff=0.1)
        group.add(sub)
    return group


def reaction_arrow(enzyme: str, energy_tag: str = None, energy_color=None) -> VGroup:
    """Flèche de réaction avec le nom de l'enzyme au-dessus,
    et un tag optionnel (ex: '+ ATP', '- ATP', '+ NADH') en dessous."""
    arrow = Arrow(UP * 0.6, DOWN * 0.6, color=WHITE, stroke_width=5, max_tip_length_to_length_ratio=0.35)
    enzyme_label = Text(enzyme, font_size=22, color=COLOR_ENZYME, slant=ITALIC)
    enzyme_label.next_to(arrow, LEFT, buff=0.25)
    group = VGroup(arrow, enzyme_label)
    if energy_tag:
        tag = Text(energy_tag, font_size=22, color=energy_color, weight=BOLD)
        tag.next_to(arrow, RIGHT, buff=0.25)
        group.add(tag)
    return group


class Intro(Scene):
    """Écran d'introduction avec le glucose de départ."""

    def construct(self):
        self.camera.background_color = "#1a1a1a"

        title = Text("La Glycolyse", font_size=56, color=WHITE, weight=BOLD)
        subtitle = Text(
            "10 étapes pour transformer\nle glucose en énergie",
            font_size=28, color=GRAY_B, line_spacing=1.2
        )
        subtitle.next_to(title, DOWN, buff=0.5)

        self.play(Write(title))
        self.play(FadeIn(subtitle, shift=UP * 0.3))
        self.wait(1.5)
        self.play(FadeOut(title), FadeOut(subtitle))

        glucose = molecule_box("Glucose", "6 carbones")
        glucose.set_color(COLOR_MOLECULE)
        self.play(FadeIn(glucose, scale=0.8))
        self.wait(0.5)

        note = Text(
            "1 molécule de glucose\n→ 2 molécules de pyruvate",
            font_size=24, color=GRAY_B, line_spacing=1.3
        )
        note.next_to(glucose, DOWN, buff=1.0)
        self.play(FadeIn(note))
        self.wait(2)
        self.play(FadeOut(glucose), FadeOut(note))


class Phase1Investissement(Scene):
    """Étapes 1 à 5 : phase d'investissement d'énergie (consommation d'ATP,
    scission de la molécule à 6 carbones en 2 molécules à 3 carbones)."""

    def construct(self):
        self.camera.background_color = "#1a1a1a"

        header = Text("Phase 1 : Investissement d'énergie", font_size=30, color=WHITE, weight=BOLD)
        header.to_edge(UP, buff=0.6)
        self.play(Write(header))
        self.wait(0.5)

        # Séquence des molécules pour les étapes 1 à 4 (avant la scission)
        steps = [
            {"name": "Glucose", "carbons": "6C", "enzyme": "Hexokinase",
             "tag": "- ATP", "tag_color": COLOR_ATP},
            {"name": "Glucose-6-\nphosphate", "carbons": "6C", "enzyme": "Phosphoglucose\nisomérase",
             "tag": None, "tag_color": None},
            {"name": "Fructose-6-\nphosphate", "carbons": "6C", "enzyme": "Phosphofructokinase",
             "tag": "- ATP", "tag_color": COLOR_ATP},
            {"name": "Fructose-1,6-\nbiphosphate", "carbons": "6C", "enzyme": "Aldolase",
             "tag": None, "tag_color": None},
        ]

        current = molecule_box(steps[0]["name"], steps[0]["carbons"])
        current.move_to(UP * 4.5)
        self.play(FadeIn(current, scale=0.8))
        self.wait(0.5)

        for i in range(1, len(steps)):
            arrow = reaction_arrow(
                steps[i - 1]["enzyme"],
                steps[i - 1]["tag"],
                steps[i - 1]["tag_color"],
            )
            arrow.next_to(current, DOWN, buff=0.3)
            self.play(GrowArrow(arrow[0]), FadeIn(arrow[1]))
            if len(arrow) > 2:
                self.play(FadeIn(arrow[2], shift=RIGHT * 0.2))
            self.wait(0.4)

            next_mol = molecule_box(steps[i]["name"], steps[i]["carbons"])
            next_mol.next_to(arrow, DOWN, buff=0.3)
            self.play(FadeIn(next_mol, shift=DOWN * 0.3))
            self.wait(1.0)

            # On regroupe et on remonte l'ensemble pour garder de la place à l'écran
            group_so_far = VGroup(current, arrow, next_mol)
            self.play(
                group_so_far.animate.shift(UP * 2.2),
                FadeOut(current), FadeOut(arrow),
                run_time=0.6
            )
            current = next_mol

        self.wait(0.5)
        self.play(current.animate.move_to(UP * 4.5))

        # Étape clé : la scission en 2 molécules à 3 carbones
        split_label = Text(
            "Aldolase coupe la molécule\nen 2 fragments à 3 carbones",
            font_size=24, color=COLOR_ENZYME, line_spacing=1.2
        )
        split_label.next_to(current, DOWN, buff=0.4)
        self.play(FadeIn(split_label))
        self.wait(1.5)
        self.play(FadeOut(split_label))

        dhap = molecule_box("DHAP", "3C")
        g3p = molecule_box("G3P", "3C")
        dhap.next_to(current, DOWN, buff=1.0).shift(LEFT * 2.0)
        g3p.next_to(current, DOWN, buff=1.0).shift(RIGHT * 2.0)

        split_arrow_l = Arrow(current.get_bottom(), dhap.get_top(), color=WHITE, stroke_width=4)
        split_arrow_r = Arrow(current.get_bottom(), g3p.get_top(), color=WHITE, stroke_width=4)

        self.play(
            GrowArrow(split_arrow_l), GrowArrow(split_arrow_r),
            FadeIn(dhap, shift=DOWN * 0.2), FadeIn(g3p, shift=DOWN * 0.2),
        )
        self.wait(0.8)

        conversion_note = Text(
            "DHAP se convertit aussi en G3P\n→ tout continue en double (x2) !",
            font_size=22, color=COLOR_NADH, line_spacing=1.2
        )
        conversion_note.next_to(VGroup(dhap, g3p), DOWN, buff=0.6)
        self.play(FadeIn(conversion_note))
        self.wait(2)

        self.play(*[FadeOut(m) for m in self.mobjects])


class Phase2Production(Scene):
    """Étapes 6 à 10 : phase de production d'énergie (x2 car on part de
    2 molécules de G3P), avec production de NADH et d'ATP."""

    def construct(self):
        self.camera.background_color = "#1a1a1a"

        header = Text("Phase 2 : Production d'énergie", font_size=30, color=WHITE, weight=BOLD)
        header.to_edge(UP, buff=0.6)
        subheader = Text("(chaque étape se produit x2)", font_size=20, color=GRAY_B)
        subheader.next_to(header, DOWN, buff=0.15)
        self.play(Write(header), FadeIn(subheader))
        self.wait(0.5)

        steps = [
            {"name": "G3P", "carbons": "3C x2", "enzyme": "G3P\ndéshydrogénase",
             "tag": "+ 2 NADH", "tag_color": COLOR_NADH},
            {"name": "1,3-biphospho-\nglycérate", "carbons": "3C x2", "enzyme": "Phosphoglycérate\nkinase",
             "tag": "+ 2 ATP", "tag_color": COLOR_ATP},
            {"name": "3-phospho-\nglycérate", "carbons": "3C x2", "enzyme": "Phosphoglycérate\nmutase",
             "tag": None, "tag_color": None},
            {"name": "2-phospho-\nglycérate", "carbons": "3C x2", "enzyme": "Énolase",
             "tag": None, "tag_color": None},
            {"name": "Phosphoénol-\npyruvate", "carbons": "3C x2", "enzyme": "Pyruvate\nkinase",
             "tag": "+ 2 ATP", "tag_color": COLOR_ATP},
        ]

        current = molecule_box(steps[0]["name"], steps[0]["carbons"])
        current.move_to(UP * 5)
        self.play(FadeIn(current, scale=0.8))
        self.wait(0.4)

        for i in range(1, len(steps)):
            arrow = reaction_arrow(
                steps[i - 1]["enzyme"],
                steps[i - 1]["tag"],
                steps[i - 1]["tag_color"],
            )
            arrow.next_to(current, DOWN, buff=0.3)
            self.play(GrowArrow(arrow[0]), FadeIn(arrow[1]))
            if len(arrow) > 2:
                self.play(FadeIn(arrow[2], shift=RIGHT * 0.2))
            self.wait(0.4)

            next_mol = molecule_box(steps[i]["name"], steps[i]["carbons"])
            next_mol.next_to(arrow, DOWN, buff=0.3)
            self.play(FadeIn(next_mol, shift=DOWN * 0.3))
            self.wait(1.0)

            group_so_far = VGroup(current, arrow, next_mol)
            self.play(
                group_so_far.animate.shift(UP * 2.3),
                FadeOut(current), FadeOut(arrow),
                run_time=0.6
            )
            current = next_mol

        self.wait(0.3)
        self.play(current.animate.move_to(UP * 5))

        # Dernière étape : phosphoénolpyruvate -> pyruvate
        final_arrow = reaction_arrow("Pyruvate kinase", "+ 2 ATP", COLOR_ATP)
        final_arrow.next_to(current, DOWN, buff=0.3)
        self.play(GrowArrow(final_arrow[0]), FadeIn(final_arrow[1]), FadeIn(final_arrow[2]))
        self.wait(0.5)

        pyruvate = molecule_box("Pyruvate", "3C x2")
        pyruvate.set_color(COLOR_PYRUVATE)
        pyruvate.next_to(final_arrow, DOWN, buff=0.3)
        self.play(FadeIn(pyruvate, shift=DOWN * 0.3))
        self.wait(2)

        self.play(*[FadeOut(m) for m in self.mobjects])


class Bilan(Scene):
    """Écran récapitulatif du bilan net de la glycolyse."""

    def construct(self):
        self.camera.background_color = "#1a1a1a"

        title = Text("Bilan net de la glycolyse", font_size=40, color=WHITE, weight=BOLD)
        title.to_edge(UP, buff=1.0)
        self.play(Write(title))
        self.wait(0.5)

        items = VGroup(
            Text("1 Glucose", font_size=30, color=COLOR_MOLECULE),
            Text("→", font_size=30, color=WHITE),
            Text("2 Pyruvate", font_size=30, color=COLOR_PYRUVATE),
        ).arrange(RIGHT, buff=0.4)
        items.move_to(UP * 2)

        atp_line = Text("Net : 2 ATP produits", font_size=30, color=COLOR_ATP, weight=BOLD)
        nadh_line = Text("2 NADH produits", font_size=30, color=COLOR_NADH, weight=BOLD)
        detail = Text(
            "(4 ATP produits - 2 ATP consommés\nau début = 2 ATP net)",
            font_size=20, color=GRAY_B, line_spacing=1.2
        )

        atp_line.next_to(items, DOWN, buff=0.8)
        nadh_line.next_to(atp_line, DOWN, buff=0.4)
        detail.next_to(nadh_line, DOWN, buff=0.5)

        self.play(FadeIn(items, shift=UP * 0.2))
        self.wait(0.5)
        self.play(FadeIn(atp_line, shift=UP * 0.2))
        self.wait(0.3)
        self.play(FadeIn(nadh_line, shift=UP * 0.2))
        self.wait(0.3)
        self.play(FadeIn(detail))
        self.wait(2.5)

        outro = Text("Le pyruvate continue ensuite\nvers le cycle de Krebs !", font_size=24, color=GRAY_B, line_spacing=1.3)
        outro.next_to(detail, DOWN, buff=1.0)
        self.play(FadeIn(outro))
        self.wait(2.5)
../../CODE_OF_CONDUCT.md
